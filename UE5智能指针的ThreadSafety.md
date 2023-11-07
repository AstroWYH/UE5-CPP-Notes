By default, Smart Pointers are only safe to access on a single thread. If you need multiple threads to have access, use the thread-safe versions of Smart Pointer classes:
默认情况下，智能指针只能在单个线程上安全访问。**如果您需要多个线程进行访问**，请使用智能指针类的线程安全版本：

- **`TSharedPtr<T, ESPMode::ThreadSafe>`**
- **`TSharedRef<T, ESPMode::ThreadSafe>`**
- **`TWeakPtr<T, ESPMode::ThreadSafe>`**
- **`TSharedFromThis<T, ESPMode::ThreadSafe>`**

由于原子引用计数，这些**线程安全版本比默认版本慢一些**，但它们的行为与常规 C++ 指针一致：

- Reads and copies are always thread-safe.
  读取和复制始终是线程安全的。
- Writes and resets must be synchronized to be safe.
  **写入和重置必须同步才能安全**。

If you know your pointer will never be accessed by more than one thread, you can get better performance by avoiding the thread-safe versions.
如果您知道您的指针永远不会被多个线程访问，则可以**通过避免线程安全版本来获得更好的性能**。

[Smart Pointers in Unreal Engine | Unreal Engine 5.3 Documentation --- 虚幻引擎中的智能指针|虚幻引擎 5.3 文档](https://docs.unrealengine.com/5.3/en-US/smart-pointers-in-unreal-engine/)