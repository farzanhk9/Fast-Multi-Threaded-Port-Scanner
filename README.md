import socket
import threading
from queue import Queue

print_lock = threading.Lock()

def scan_port(target, poort):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)
        result = s.connect_ex((target, poort))
        if result == 0:
            with print_lock:
                print(f"[OPEN] Port {port}")
        s.close()
    except:
        pass

def worker(target, queue):
    while not queue.empty():
        port = queue.get()
        scan_port(target, port)
        queue.task_done()

def main():
    target = input("Enter target IP or domain: ").strip()
    
    try:
        target_ip = socket.gethostbyname(target)
    except socket.gaierror:
        print("Invalid target.")
        return

    start_port = int(input("Start port: "))
    end_port = int(input("End port: "))

    print(f"\nScanning {target_ip} from port {start_port} to {end_port}...\n")

    queue = Queue()

    for port in range(start_port, end_port + 1):
        queue.put(port)

    thread_count = 100

    for _ in range(thread_count):
        t = threading.Thread(target=worker, args=(target_ip, queue))
        t.daemon = True
        t.start()

    queue.join()

    print("\nScan complete.")

if __name__ == "__main__":
    main()
