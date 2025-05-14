import threading
import random


shared_memory = {"x": 0}


cache_traffic = {"with": 0, "without": 0}
coherence_messages = 0


NUM_THREADS = 4
NUM_OPERATIONS = 100


def thread_no_coherence(thread_id):
    local_cache = {}  # cache lokal untuk thread ini
    for _ in range(NUM_OPERATIONS):
        if random.random() < 0.5:
            # Operasi READ
            if "x" not in local_cache:
                local_cache["x"] = shared_memory["x"]
                cache_traffic["without"] += 1  # akses memori
            _ = local_cache["x"]  # baca dari cache lokal
        else:
            # Operasi WRITE
            local_cache["x"] = thread_id
            shared_memory["x"] = thread_id
            cache_traffic["without"] += 1  # tulis ke memori


def thread_with_coherence(thread_id, cache_states, lock):
    global coherence_messages
    local_cache = {}  # cache lokal untuk thread ini

    for _ in range(NUM_OPERATIONS):
        op = random.choice(["read", "write"])  # pilih operasi secara acak

        with lock:  # pastikan akses memori bersama sinkron
            if op == "read":

                if cache_states[thread_id] == "I":
                    local_cache["x"] = shared_memory["x"]
                    cache_states[thread_id] = "S"
                    cache_traffic["with"] += 1
                    coherence_messages += 1  
                _ = local_cache["x"]  

            else:  
               
                for i in range(NUM_THREADS):
                    if i != thread_id and cache_states[i] != "I":
                        cache_states[i] = "I"
                        coherence_messages += 1  

                
                shared_memory["x"] = thread_id
                local_cache["x"] = thread_id
                cache_states[thread_id] = "M"
                cache_traffic["with"] += 1  


def run_simulation():
    global shared_memory, cache_traffic, coherence_messages

   
    shared_memory = {"x": 0}
    cache_traffic = {"with": 0, "without": 0}
    coherence_messages = 0

    print("Simulasi tanpa koherensi...")
    threads = []
    for i in range(NUM_THREADS):
        t = threading.Thread(target=thread_no_coherence, args=(i,))
        threads.append(t)
        t.start()
    for t in threads:
        t.join()

    print(f"Total cache traffic tanpa koherensi: {cache_traffic['without']}")
    print()

    print("Simulasi dengan koherensi (MSI)...")
    shared_memory = {"x": 0}  
    cache_states = ["I"] * NUM_THREADS  
    lock = threading.Lock() 
    threads = []
    for i in range(NUM_THREADS):
        t = threading.Thread(target=thread_with_coherence, args=(i, cache_states, lock))
        threads.append(t)
        t.start()
    for t in threads:
        t.join()

    print(f"Total cache traffic dengan koherensi: {cache_traffic['with']}")
    print(f"Total pesan koherensi: {coherence_messages}")
    print()

    
    print("Perbandingan:")
    print(f"Cache Traffic (Tanpa Koherensi): {cache_traffic['without']}")
    print(f"Cache Traffic (Dengan Koherensi): {cache_traffic['with']}")
    print(f"Pesan Koherensi yang Dikirim: {coherence_messages}")


if __name__ == "__main__":
    run_simulation()
