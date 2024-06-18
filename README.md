# Multilevel-Feedback-Scheduling

 Class Aufgabe:
    def __init__(self, name, laufzeit, ankunftszeit):
        # Initialisierungsmethode, die beim Erstellen einer neuen Aufgabe aufgerufen wird
        self.name = name  # Name der Aufgabe
        self.laufzeit = laufzeit  # Gesamtzeit, die die Aufgabe für die Ausführung benötigt
        self.ankunftszeit = ankunftszeit  # Zeitpunkt, zu dem die Aufgabe ins System eintritt
        self.verbleibende_zeit = laufzeit  # Verbleibende Zeit, die die Aufgabe zur Ausführung benötigt
        self.startzeit = None  # Zeitpunkt, zu dem die Aufgabe erstmals auf der CPU ausgeführt wird
        self.endzeit = None  # Zeitpunkt, zu dem die Aufgabe ihre Ausführung abschließt
        self.wartezeit = 0  # Gesamtzeit, die die Aufgabe im Wartezustand verbringt

    def __repr__(self):
        # Methode zur Darstellung der Aufgabe als String (nützlich für Debug-Zwecke)
        return f"Aufgabe(name={self.name}, laufzeit={self.laufzeit}, ankunftszeit={self.ankunftszeit})"

# Klasse, die eine Warteschlange im System repräsentiert
class AufgabenWarteschlange:
    def __init__(self, zeitquantum):
        # Initialisierungsmethode, die beim Erstellen einer neuen Warteschlange aufgerufen wird
        self.zeitquantum = zeitquantum  # Zeitquantum für die Round-Robin-Scheduling
        self.elemente = []  # Liste zur Speicherung der Aufgaben in der Warteschlange

    def hinzufuegen(self, element):
        # Methode zum Hinzufügen einer Aufgabe zur Warteschlange
        self.elemente.append(element)  # Fügt die Aufgabe am Ende der Warteschlange hinzu

    def entfernen(self):
        # Methode zum Entfernen und Zurückgeben der ersten Aufgabe in der Warteschlange
        if not self.ist_leer():  # Überprüft, ob die Warteschlange nicht leer ist
            return self.elemente.pop(0)  # Entfernt und gibt die erste Aufgabe in der Warteschlange zurück
        else:
            return None  # Gibt None zurück, wenn die Warteschlange leer ist

    def ist_leer(self):
        # Methode zum Überprüfen, ob die Warteschlange leer ist
        return len(self.elemente) == 0  # Gibt True zurück, wenn die Warteschlange leer ist, andernfalls False

    def groesse(self):
        # Methode zur Rückgabe der Anzahl der Aufgaben in der Warteschlange
        return len(self.elemente)  # Gibt die Länge der Liste zurück

    def __repr__(self):
        # Methode zur Darstellung der Warteschlange als String (nützlich für Debug-Zwecke)
        return f"AufgabenWarteschlange(zeitquantum={self.zeitquantum}, elemente={self.elemente})"

# Klasse, die das Multilevel-Feedback-Scheduling-System repräsentiert
class MehrstufigeFeedbackWarteschlange:
    def __init__(self, zeitquantums):
        # Initialisierungsmethode, die beim Erstellen eines neuen Scheduling-Systems aufgerufen wird
        self.warteschlangen = [AufgabenWarteschlange(zq) for zq in zeitquantums]  # Erstellt eine Liste von Warteschlangen basierend auf den Zeitquantums
        self.aktuelle_zeit = 0  # Aktuelle Systemzeit

    def aufgabe_hinzufuegen(self, aufgabe):
        # Methode zum Hinzufügen einer neuen Aufgabe zur ersten Warteschlange
        self.warteschlangen[0].hinzufuegen(aufgabe)  # Fügt die Aufgabe in die erste (höchste) Warteschlange ein

    def ausfuehren(self):
        # Methode zum Ausführen der Aufgaben gemäß dem Multilevel-Feedback-Scheduling-Algorithmus
        while any(not w.ist_leer() for w in self.warteschlangen):  # Solange es nicht leere Warteschlangen gibt
            for i, warteschlange in enumerate(self.warteschlangen):  # Durchläuft alle Warteschlangen
                if not warteschlange.ist_leer():  # Überprüft, ob die aktuelle Warteschlange nicht leer ist
                    aufgabe = warteschlange.entfernen()  # Entfernt die erste Aufgabe aus der Warteschlange
                    if aufgabe.startzeit is None:  # Überprüft, ob die Aufgabe zum ersten Mal ausgeführt wird
                        aufgabe.startzeit = self.aktuelle_zeit  # Setzt die Startzeit der Aufgabe auf die aktuelle Systemzeit

                    ausfuehrungszeit = min(warteschlange.zeitquantum, aufgabe.verbleibende_zeit)  # Berechnet die Ausführungszeit (kleineres von Zeitquantum und verbleibender Zeit)
                    aufgabe.verbleibende_zeit -= ausfuehrungszeit  # Verringert die verbleibende Ausführungszeit der Aufgabe
                    self.aktuelle_zeit += ausfuehrungszeit  # Erhöht die aktuelle Systemzeit um die Ausführungszeit

                    if aufgabe.verbleibende_zeit > 0:  # Überprüft, ob die Aufgabe noch nicht fertig ist
                        if i < len(self.warteschlangen) - 1:  # Überprüft, ob es eine tiefere Warteschlange gibt
                            self.warteschlangen[i + 1].hinzufuegen(aufgabe)  # Fügt die Aufgabe in die nächsttiefere Warteschlange ein
                        else:
                            warteschlange.hinzufuegen(aufgabe)  # Fügt die Aufgabe zurück in die gleiche Warteschlange ein, wenn es keine tiefere gibt
                    else:
                        aufgabe.endzeit = self.aktuelle_zeit  # Setzt die Endzeit der Aufgabe auf die aktuelle Systemzeit
                        aufgabe.wartezeit = aufgabe.endzeit - aufgabe.ankunftszeit - aufgabe.laufzeit  # Berechnet die Wartezeit der Aufgabe

                    print(f'Aufgabe ausgeführt: {aufgabe.name} | Zeit: {self.aktuelle_zeit} | Verbleibende Zeit: {aufgabe.verbleibende_zeit}')


