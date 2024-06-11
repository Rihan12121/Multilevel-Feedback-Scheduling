# Multilevel-Feedback-Scheduling

 class Aufgabe:
    def __init__(self, name, laufzeit, ankunftszeit):
       
       
        self.name = name 
        self.laufzeit = laufzeit  
        self.ankunftszeit = ankunftszeit  
        self.verbleibende_zeit = laufzeit 
        self.startzeit = None  
        self.endzeit = None  
        self.wartezeit = 0 

    def __repr__(self):
          
        return f"Aufgabe(name={self.name}, laufzeit={self.laufzeit}, ankunftszeit={self.ankunftszeit})"
