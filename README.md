# Multilevel-Feedback-Scheduling

from collections import deque
from typing import List

class Task:
    def __init__(self, name: str, burst_time: float, arrival_time: float):
        self.name = name  # Name des Prozesses
        self.burst_time = burst_time  # CPU-Laufzeit des Prozesses
        self.arrival_time = arrival_time  # Ankunftszeit des Prozesses
        self.remaining_time = burst_time  # Verbleibende CPU-Zeit des Prozesses
        self.start_time = None  # Startzeit des Prozesses
        self.end_time = None  # Endzeit des Prozesses
        self.waiting_time = 0.0  # Wartezeit des Prozesses

    def __repr__(self):
        return (f"Task(name={self.name}, burst_time={self.burst_time}, "
                f"arrival_time={self.arrival_time}, remaining_time={self.remaining_time})")

class Queue:
    def __init__(self, time_quantum: float):
        self.time_quantum = time_quantum  # Zeitquantum für diese Warteschlange
        self.tasks = deque()  # Doppelt verkettete Liste zur Speicherung der Prozesse

    def add(self, task: Task):
        self.tasks.append(task)  # Fügt einen Prozess am Ende der Warteschlange hinzu

    def remove(self) -> Task:
        if self.tasks:
            return self.tasks.popleft()  # Entfernt und gibt den ersten Prozess in der Warteschlange zurück

    def is_empty(self) -> bool:
        return len(self.tasks) == 0  # Überprüft, ob die Warteschlange leer ist

    def size(self) -> int:
        return len(self.tasks)  # Gibt die Anzahl der Prozesse in der Warteschlange zurück

### Logik für das Verschieben von Prozessen zwischen den Warteschlangen

class MultilevelFeedbackQueue:
    def __init__(self, time_quantums: List[float]):
        self.queues = [Queue(tq) for tq in time_quantums]  # Erstellen von Warteschlangen mit unterschiedlichen Zeitquantums
        self.current_time = 0.0  # Aktuelle Zeit im Scheduler

    def add_task(self, task: Task):
        self.queues[0].add(task)  # Fügt den Prozess der ersten Warteschlange hinzu

    def move_task_to_next_queue(self, task: Task, current_queue_index: int):
        if current_queue_index < len(self.queues) - 1:
            self.queues[current_queue_index + 1].add(task)  # Verschiebt den Prozess in die nächste Warteschlange

    def execute(self):
        while any(not q.is_empty() for q in self.queues):
            for i, queue in enumerate(self.queues):
                if not queue.is_empty():
                    task = queue.remove()
                    if task.start_time is None:
                        task.start_time = self.current_time  # Setzt die Startzeit des Prozesses
                    execution_time = min(queue.time_quantum, task.remaining_time)  # Bestimmt die Ausführungszeit
                    task.remaining_time -= execution_time  # Aktualisiert die verbleibende Zeit
                    self.current_time += execution_time  # Aktualisiert die aktuelle Zeit
                    if task.remaining_time > 0:
                        self.move_task_to_next_queue(task, i)  # Verschiebt den Prozess in die nächste Warteschlange
                    else:
                        task.end_time = self.current_time  # Setzt die Endzeit des Prozesses
                        task.waiting_time = task.end_time - task.arrival_time - task.burst_time  # Berechnet die Wartezeit
                        print(f"Task executed: {task.name} | Time: {self.current_time} | Remaining Time: {task.remaining_time}")

if __name__ == "__main__":
    # Beispieldaten
    tasks = [
        Task("T1", 24, 0),
        Task("T2", 3, 0),
        Task("T3", 3, 0),
        Task("T4", 6, 5),
        Task("T5", 2, 10),
    ]

    time_quantums = [4, 8, 16]  # Zeitquantums für die verschiedenen Warteschlangen

    # Instanziierung des Multilevel-Feedback-Queue-Systems
    mfq = MultilevelFeedbackQueue(time_quantums)

    # Hinzufügen der Aufgaben (sortiert nach Ankunftszeit)
    for task in sorted(tasks, key=lambda t: t.arrival_time):
        mfq.add_task(task)

    # Ausführung des Scheduling-Algorithmus
    mfq.execute()





             
