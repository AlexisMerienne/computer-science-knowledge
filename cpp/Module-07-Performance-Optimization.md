 # Module 07 — Performance Optimization

 Objective: Learn techniques to optimize C++ code for performance.

 Contents

 - [7.1 Profiling tools (perf, Valgrind, VTune)](#71-profiling-tools-perf-valgrind-vtune)
 - [7.2 Algorithmic optimization and data structure selection](#72-algorithmic-optimization-and-data-structure-selection)
 - [7.3 Cache-aware programming and data locality](#73-cache-aware-programming-and-data-locality)
 - [7.4 Multithreading and concurrency (C++11 threads, std::async, atomics, locks)](#74-multithreading-and-concurrency)

 ---

 ## 7.1 Profiling tools

 - Linux: perf, Valgrind (callgrind), gprof, Intel VTune; Windows: Visual Studio Profiler, Windows Performance Analyzer.
 - Make small reproducible benchmarks and profile them; optimize hotspots, not what's already fast.

 Tip: measure before optimizing.

 ---
 ## 7.2 Algorithmic optimization and data structure selection

 - Time complexity and big-O matter: prefer a good algorithm over micro-optimizations.
 - Use appropriate containers: unordered_map when hash access is appropriate; vector for sequential data and cache locality.

 Example: choose right structure

 ```cpp
 #include <vector>
 #include <unordered_map>
 #include <iostream>

 // small demonstration: reserve memory to improve insertion speed
 int main(){
     std::unordered_map<int,int> m;
     m.reserve(2000);
     for (int i=0;i<1000;i++) m[i]=i*2;
     std::cout << m.size() << '\n';
 }
 ```

 ---
 ## 7.3 Cache-aware programming and data locality

 - Structure arrays for locality (AoS vs SoA depending on access patterns).
 - Avoid unnecessary indirections and pointer chasing in tight loops.

 Example — SoA vs AoS tradeoff sketch

 ```cpp
 struct Particle { float x,y,z; float vx,vy,vz; };
 // If only positions are read sequentially, store positions in separate arrays for better locality.
 ```

 ---
 ## 7.4 Multithreading and concurrency

 - C++11 provides std::thread, std::mutex, std::atomic, std::future/std::async for high-level constructs.
 - Avoid data races: prefer message passing, thread-safe queues, or immutability for sharing.

 Example — safe producer/consumer using a queue + condition_variable

 ```cpp
 #include <queue>
 #include <mutex>
 #include <condition_variable>
 #include <thread>
 #include <iostream>

 std::queue<int> q; std::mutex m; std::condition_variable cv; bool done = false;

 void producer(){ for(int i=0;i<5;i++){ std::lock_guard<std::mutex> lk(m); q.push(i); cv.notify_one(); }}
 void consumer(){
     while(!done){
         std::unique_lock<std::mutex> lk(m);
         cv.wait(lk, []{ return !q.empty() || done; });
         while(!q.empty()){ std::cout << q.front() << '\n'; q.pop(); }
     }
 }

 int main(){
     std::thread p(producer);
     std::thread c(consumer);
     p.join();
     { std::lock_guard<std::mutex> lk(m); done = true; cv.notify_one(); }
     c.join();
 }
 ```

 ---

 Continue to Module 08 — Testing and Debugging to enforce correctness and stability.

 [Back to Overview](00-Overview.md)
 [Prev: Module 06 — Software Architecture](Module-06-Software-Architecture.md)
 [Next: Module 08 — Testing and Debugging](Module-08-Testing-Debugging.md)
