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

  const fourthWalVersionCode =
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
                std.log.err("Error! Wal synchronization failed: {any}", .{e});
                return WalError.SynchronizationError;
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
    May 1, 2026
  </Paragraph>
  <Paragraph class="text-2xl md:text-3xl font-semibold tracking-tight">
    WAL: an ongoing story
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    One of the main design ideas behind Sklad is to use lock-free data structures. Currently, the only place where locking is used is
    when data is written to the write-ahead log.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    Sklad uses a <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">Wal</code> struct that manages the write-ahead log file.
    It exposes a <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">writeRecord</code> method that writes data to the file.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    The original implementation of the method was very straightforward:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{firstWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    You can spot a few issues with this implementation. The main one is that it didn't synchronize writes with the underlying filesystem.
    The code does work, in the sense that the database can operate that way. But without durability or crash recovery guarantees, the WAL loses most of its point.
    Also, since there are no <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">fsync</code> calls, the performance is pretty good.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    To measure request latencies, a <a href="https://github.com/sklad-dev/overload" target="_blank" rel="noopener noreferrer" class="underline underline-offset-2 text-blue-700 hover:text-blue-500">custom Go tool</a>
    was used. The database was tested using <span class="font-bold">32 parallel connections</span>, sending <span class="font-bold">5 million requests</span> in
    total with a <span class="font-bold">90/10 write-to-read</span> ratio. The results for the initial <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">writeRecord</code> version are:
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
    Not bad, but the changes still need to be synced. The method was updated accordingly, and performance was measured again.
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
    At that point, it was tempting to treat this as the cost of durability. After all, the data still has to be synchronized, so what else could be improved?
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    Well, one option is to use <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">pwrite</code>
    along with an atomic variable to track the current file offset. The next iteration of the method implementation looked like this:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{thirdWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    As you can see, there is an atomic <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">u64</code> variable
    to track the last synchronized offset and an atomic <code class="bg-gray-200 px-1.5 py-0.5 rounded font-mono text-sm">bool</code>
    flag to ensure that only one thread performs synchronization at a time.
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
    Much better. But there is still a durability issue with this implementation. Consider the following scenario:
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
    In the end, a mutex is still needed for WAL synchronization. But the mutex only needs to guard the parts where threads compete to decide who performs the sync:
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
<pre class="overflow-x-auto bg-gray-200 px-6 py-3 rounded text-sm leading-relaxed font-mono"><code>{fourthWalVersionCode}</code></pre>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    In this implementation, a thread waits for other threads writing at smaller offsets to finish before attempting to acquire the synchronization lock.
    Its performance is on par with the original implementation without proper synchronization, and no durability issues or WAL corruption have been observed so far.
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
    This is where the WAL stands for now. The next step is to focus on recovery guarantees rather than latency numbers:
    record framing, checksums, and clearer handling of sync failures.
  </Paragraph>
  <GithubButton class="w-full flex justify-center mt-8 mb-2" />
</div>
