# Teleoperacja Kinova Gen3 ze zmiennymi układami odniesienia

## Opis
Projekt implementuje system teleoperacji robota Kinova Gen3 ze sprzężeniem wizyjnym w środowisku MuJoCo. Głównym celem jest redukcja obciążenia poznawczego operatora poprzez umożliwienie dynamicznego przełączania układów współrzędnych (baza robota vs. kamera typu Eye-in-Hand) podczas wykonywania precyzyjnych zadań manipulacyjnych.

### Sterowanie
- Implementacja węzła joy
- Obsługa trzech trybów sterowania:
  - **Base Frame** – sterowanie względem bazy robota
  - **Tool Frame** – sterowanie względem końcówki robota
  - **Joint Control** – bezpośrednie sterowanie złączami

### Kinematyka
- Integracja z frameworkiem MoveIt
- Rozwiązywanie kinematyki odwrotnej (IK)

### Pętla symulacji
- Przekazywanie docelowych pozycji złącz do MuJoCo
- Odbiór obrazu z wirtualnych kamer

## Architektura systemu

### Input Node
- Pobiera komendy z pada
- Realizuje transformacje macierzowe między układami odniesienia
- Generuje i wysyła wektory zadane

### MoveIt Core
- Rozwiązuje kinematykę odwrotną

### MuJoCo Node
- Symuluje fizykę i dynamikę robota
- Dostarcza sprzężenie zwrotne w postaci obrazu wideo

## Cele (Milestone 1)

### Integracja
- Wczytanie modelu manipulatora do środowiska MuJoCo
- Konfiguracja symulacji
- Podstawowe sterowanie z poziomu terminala
Projekt badawczo-rozwojowy – teleoperacja robotów manipulacyjnych.

