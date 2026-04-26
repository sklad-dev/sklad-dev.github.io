<script>
  import GithubButton from '../../lib/GithubButton.svelte';
  import Paragraph from '../../lib/Paragraph.svelte';

  const firstWalVersionCode =
`pub fn writeRecord(self: *Wal, record: *const StorageRecord) !void {
  if (!tryLockFor(&self.lock, 200)) return ApplicationError.ExecutionTimeout;
  defer self.lock.unlock();
  const max_position = try self.file.getEndPos();
  var writer = self.file.writer(&[0]u8{});

  try writer.seekTo(max_position);
  try record.write(&writer);
}`;

  const secondWalVersionCode =
`pub fn writeRecord(self: *Wal, record: *const StorageRecord) !void {
    if (!tryLockFor(&self.lock, 200)) return ApplicationError.ExecutionTimeout;
    defer self.lock.unlock();

    const max_position = try self.file.getEndPos();
    var writer = self.file.writer(&[0]u8{});

    try writer.seekTo(max_position);
    try record.write(&writer);
    try self.file.sync();
}`;

  const thirdWalVersionCode =
`pub fn writeRecord(self: *Wal, record: *const StorageRecord) !void {
    const record_size = record.sizeInMemory();
    const offset = self.eof_offset.fetchAdd(record_size, .seq_cst);

    const buffer = try self.allocator.alloc(u8, record_size);
    defer self.allocator.free(buffer);

    record.writeToBuffer(buffer, 0);
    try self.file.pwriteAll(buffer, offset);

    const my_end_offset = offset + record_size;
    if (self.synced_offset.load(.acquire) <= my_end_offset) {
        if (!self.is_flushing.swap(true, .acq_rel)) {
            defer self.is_flushing.store(false, .release);
            try self.file.sync();
            self.synced_offset.store(self.eof_offset.load(.acquire), .release);
        }
    }
}`;

  const forthWalVersionCode =
`pub fn writeRecord(self: *Wal, record: *const StorageRecord) !void {
    const record_size = record.sizeInMemory();

    var buffer: [1024]u8 = undefined;
    const slice = if (record_size <= 1024) buffer[0..record_size] else try self.allocator.alloc(u8, record_size);
    defer if (record_size > 1024) self.allocator.free(slice);

    record.writeToBuffer(slice, 0);

    const offset = self.eof_offset.fetchAdd(record_size, .acq_rel);
    const end_offset = offset + record_size;
    self.file.pwriteAll(slice, offset) catch |e| {
        _ = self.written_offset.rmw(.Max, end_offset, .seq_cst);
        return e;
    };

    while (self.written_offset.load(.acquire) < offset) {
        std.Thread.yield() catch {};
    }
    _ = self.written_offset.rmw(.Max, end_offset, .seq_cst);

    self.sync_mutex.lock();
    while (self.synced_offset.load(.acquire) < end_offset) {
        if (!self.is_flushing.swap(true, .acq_rel)) {
            const ready_to_sync = self.written_offset.load(.acquire);
            self.sync_mutex.unlock();

            self.file.sync() catch |e| {
                self.sync_mutex.lock();
                self.is_flushing.store(false, .release);
                self.sync_cond.broadcast();
                self.sync_mutex.unlock();
                std.log.err("Error! Wal syncronization failed: {any}", .{e});
                return WalError.SyncronizationError;
            };

            self.sync_mutex.lock();
            self.synced_offset.store(ready_to_sync, .release);
            self.is_flushing.store(false, .release);
            self.sync_cond.broadcast();
        } else {
            self.sync_cond.wait(&self.sync_mutex);
        }
    }
    self.sync_mutex.unlock();
}`;
</script>

<div class="w-full flex flex-col items-start mx-auto max-w-3xl mt-2 mb-20">
  <Paragraph leading='relaxed' class="mb-6">
    <a href="#/posts" class="hover:underline hover:text-gray-700 transition-colors">
      &#8592; All posts
    </a>
  </Paragraph>
  <Paragraph class="text-sm text-gray-700 font-medium" leading='relaxed'>
    April 12, 2026
  </Paragraph>
  <Paragraph class="text-2xl md:text-3xl font-semibold tracking-tight">
    WAL: an ongoing story
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    One of the main design goals of Sklad is to rely as much as possible on lock-free data structures and algorithms.
    Currently, the only place where I do use locking is when I write data to the write-ahead log.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    I have a <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">Wal</code> struct that manages the write-ahead log file.
    It exposes a <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">writeRecord</code> method that writes data to the file.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    The original implementation of the method was very straightforward:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{firstWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    You can spot a few issues with this implementation. The main one is that I wasn't synchronizing writes with the underlying filesystem.
    But it works! If you don't care much about durability and crash recovery, it works. And since there are no
    <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">fsync</code> calls, the performance is pretty decent.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    I wrote a tool in Go to measure request latencies. I tested the database using <span class="font-bold">32 parallel connections</span>,
    sending <span class="font-bold">5 million requests</span> total with a <span class="font-bold">90/10 write-to-read</span> requests ratio.
    The results for the initial <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">writeRecord</code> verison were:
  </Paragraph>
  <div class="px-8 mb-4 w-full">
    <table class="w-full text-left text-sm">
      <thead>
        <tr class="border-b border-gray-200 bg-gray-100">
          <th class="px-3 py-2 font-extrabold">p50, ms</th>
          <th class="px-3 py-2 font-extrabold">p95, ms</th>
          <th class="px-3 py-2 font-extrabold">p99, ms</th>
          <th class="px-3 py-2 font-extrabold">Throughput, req/s</th>
        </tr>
      </thead>
      <tbody>
        <tr class="border-b border-gray-100 align-top">
          <td class="px-3 py-3">1.137</td>
          <td class="px-3 py-3">1.76</td>
          <td class="px-3 py-3">2.14</td>
          <td class="px-3 py-3">26527</td>
        </tr>
      </tbody>
    </table>
  </div>
  <Paragraph class="text-base" leading='relaxed'>
    Not bad, but I still need to sync the changes. So I updated the method accordingly and measured performance again.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{secondWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    As expected, the latencies increased significantly.
  </Paragraph>
  <div class="px-8 mb-4 w-full">
    <table class="w-full text-left text-sm">
      <thead>
        <tr class="border-b border-gray-200 bg-gray-100">
          <th class="px-3 py-2 font-extrabold">p50, ms</th>
          <th class="px-3 py-2 font-extrabold">p95, ms</th>
          <th class="px-3 py-2 font-extrabold">p99, ms</th>
          <th class="px-3 py-2 font-extrabold">Throughput, req/s</th>
        </tr>
      </thead>
      <tbody>
        <tr class="border-b border-gray-100 align-top">
          <td class="px-3 py-3">2.244</td>
          <td class="px-3 py-3">4.487</td>
          <td class="px-3 py-3">12.506</td>
          <td class="px-3 py-3">11674</td>
        </tr>
      </tbody>
    </table>
  </div>
  <Paragraph class="text-base" leading='relaxed'>
    And at first I thought, well, that's just the cost of durability.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    But then I realized I could use <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">pwrite</code>
    along with an atomic variable to track the current file offset. So I updated the implementation to:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{thirdWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    As you can see, I also added an atomic <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">u64</code> to track
    the last synchronized offset and an atomic <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">bool</code> flag
    to ensure that only one thread performs synchronization at a time.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    How did this affect performance?
  </Paragraph>
  <div class="px-8 mb-4 w-full">
    <table class="w-full text-left text-sm">
      <thead>
        <tr class="border-b border-gray-200 bg-gray-100">
          <th class="px-3 py-2 font-extrabold">p50, ms</th>
          <th class="px-3 py-2 font-extrabold">p95, ms</th>
          <th class="px-3 py-2 font-extrabold">p99, ms</th>
          <th class="px-3 py-2 font-extrabold">Throughput, req/s</th>
        </tr>
      </thead>
      <tbody>
        <tr class="border-b border-gray-100 align-top">
          <td class="px-3 py-3">0.949</td>
          <td class="px-3 py-3">1.516</td>
          <td class="px-3 py-3">1.893</td>
          <td class="px-3 py-3">32111</td>
        </tr>
      </tbody>
    </table>
  </div>
  <Paragraph class="text-base" leading='relaxed'>
    Much better! But there's still a durability issue with this implementation. Consider the following scenario:
  </Paragraph>
  <ol type=1 start=1 class="leading-relaxed text-base px-16 mb-4 list-decimal ml-4">
    <li><span class="font-bold">Thread 1</span> starts <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">self.file.sync()</code></li>
    <li><span class="font-bold">Thread 2</span> finishes <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">pwriteAll</code> while <span class="font-bold">Thread 1</span> is still syncing </li>
    <li><span class="font-bold">Thread 2</span> checks <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">is_flushing</code>, sees it is <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">true</code>, and immediately returns</li>
    <li>The database reports success to the client</li>
    <li>The system crashes (e.g., power loss)</li>
  </ol>
  <Paragraph class="text-base" leading='relaxed'>
    The client believes the data was safely persisted, but on restart, the database won't be able to recover it.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    So in the end, I decided to keep a mutex for WAL synchronization. But the mutex can lock just the parts where threads compete to decide who performs the sync:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{forthWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    In this implementation, a thread will wait for other threads writing at smaller offsets to finish before attempting to acquire the synchronization lock.
    Its performance is on par with the original one without proper synchronization, and so far I haven't spotted any durability issues or WAL corruption.
    Here are the latest latency numbers:
  </Paragraph>
  <div class="px-8 mb-4 w-full">
    <table class="w-full text-left text-sm">
      <thead>
        <tr class="border-b border-gray-200 bg-gray-100">
          <th class="px-3 py-2 font-extrabold">p50, ms</th>
          <th class="px-3 py-2 font-extrabold">p95, ms</th>
          <th class="px-3 py-2 font-extrabold">p99, ms</th>
          <th class="px-3 py-2 font-extrabold">Throughput, req/s</th>
        </tr>
      </thead>
      <tbody>
        <tr class="border-b border-gray-100 align-top">
          <td class="px-3 py-3">1.356</td>
          <td class="px-3 py-3">1.869</td>
          <td class="px-3 py-3">2.193</td>
          <td class="px-3 py-3">22982</td>
        </tr>
      </tbody>
    </table>
  </div>
  <Paragraph class="text-base" leading='relaxed'>
    This is the implementation I have right now. I'm planning to take a short break from performance experiments and then focus on a few robustness improvements:
  </Paragraph>
  <ol type=1 start=1 class="leading-relaxed text-base px-16 mb-4 list-decimal ml-4">
    <li>I need to prepend each WAL record with its length and add a per-record checksum for crash recovery.</li>
    <li>I want to add a failed sync counter to prevent cascading retries on sync failure.</li>
  </ol>
  <GithubButton class="w-full flex justify-center mt-8 mb-2" />
</div>
