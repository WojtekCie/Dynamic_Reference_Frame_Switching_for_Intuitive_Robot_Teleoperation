# Advanced Teleoperation Control for Redundant Manipulators (Kinova Gen3)

## 📌 Project Overview / Opis Projektu
Projekt skupia się na stworzeniu zaawansowanego algorytmu teleoperacji dla manipulatorów robotycznych (6-DOF oraz 7-DOF Kinova Gen3). Głównym celem jest redukcja obciążenia poznawczego (cognitive load) operatora oraz poprawa precyzji w skomplikowanych zadaniach manipulacyjnych realizowanych na odległość. 

System implementuje zamkniętą pętlę kinematyki odwrotnej (**CLIK**), dynamiczną transformację układów odniesienia w locie (**Reference Frame Switching**) oraz matematyczne omijanie osobliwości za pomocą metody **Damped Least Squares (DLS)**.

## 📖 Theoretical Background & Motivation

### 1. Przewaga konfiguracji Eye-in-Hand w teleoperacji precyzyjnej
W zadaniach wymagających wysokiej precyzji i wchodzenia w bezpośredni kontakt ze środowiskiem (np. *peg-in-hole*, obsługa paneli, chwytanie nieregularnych obiektów), teleoperacja osiąga najlepsze wyniki przy zastosowaniu kamery w konfiguracji **Eye-in-Hand** (kamera zamontowana bezpośrednio na efektorze końcowym). Konfiguracja ta pozwala na całkowite uniknięcie problemu okluzji (zasłaniania widoku przez ramię robota) i umożliwia niezwykle intuicyjne sterowanie oraz obserwację środowiska pod różnymi kątami.

### 2. Problem niedopasowania układów współrzędnych (Camera Frame Misalignment)
Mimo zalet kamery na chwytaku, bezpośrednie wdrożenie podglądu *Eye-in-Hand* przy pozostawieniu układu sterowania (joysticka) w przestrzeni bazy robota (*Robot Base Frame*) prowadzi do drastycznej degradacji wyników. 
Jak dobitnie udowadniają autorzy w artykule *"An Experimental Study on the Effects of Control Frame and View in Robot Teleoperation"*, niedopasowanie przestrzeni wizyjnej do przestrzeni sterowania (tzw. *visual-motor misalignment*) wywołuje ogromną dezorientację. Operator musi w czasie rzeczywistym dokonywać w głowie transformacji macierzy rotacji, co prowadzi do częstych kolizji. Zjawisko to potwierdzają również nowsze badania: brak korekty układu odniesienia dla kamery *Eye-in-Hand* wydłuża czas wykonania zadania, zwiększa drgania (unsteadiness) o niemal 50% i podnosi stres mentalny operatora o ponad 60%.

### 3. Rozwiązanie: Dynamic Reference Frame Switching
Aby rozwiązać powyższy problem, w projekcie wdrożono algorytm dynamicznej transformacji układu odniesienia ze sztywnej ramy robota (*Robot Frame*) na przestrzeń widzenia kamery (*View/Tool Frame*). Zastosowane podejście wpisuje się w najnowsze paradygmaty **Shared Control** (współdzielenia kontroli). Inspirując się architekturami przedstawionymi m.in. w publikacji z konferencji IEEE ICRA (*"Interactive Teleoperation Interface for Semi-autonomous Control of Robot Arms"*), system rozdziela zadania: człowiek decyduje o wektorach i kierunku w intuicyjnej dla siebie przestrzeni wizyjnej, a algorytm w locie przelicza wymagane transformacje (Twist) na bazę robota, gwarantując bezpieczny, płynny ruch.

---

## ⚙️ System Architecture

Architektura kontrolera bazuje na dwóch trybach, które operator może przełączać w locie:

* **Base Frame Mode (Przestrzeń Bazy):** Polecenia prędkości z joypada są interpretowane względem globalnej podstawy robota. Tryb optymalny do makro-ruchów (np. szybkie zbliżenie ramienia w rejon obszaru roboczego na podstawie widoku z kamery zewnętrznej).
* **Tool Frame / View Mode (Przestrzeń Efektora):** Polecenia z joypada są matematycznie transformowane (z użyciem macierzy rotacji $R_{tool}^{base}$) do przestrzeni kamery *Eye-in-Hand*. Ruch w przód na joypadzie zawsze oznacza ruch "w głąb ekranu", niezależnie od aktualnego wygięcia ramienia. Tryb kluczowy do precyzyjnych zadań kontaktowych.

### Warstwa bezpieczeństwa matematycznego
1. **CLIK (Closed-Loop Inverse Kinematics):** Aby zniwelować dryf numeryczny wynikający z całkowania prędkości, system został wyposażony w sprzężenie zwrotne kontrolujące uchyb pozycji w otwartej przestrzeni kartezjańskiej.
2. **DLS (Damped Least Squares):** W celu zabezpieczenia sprzętu przed nieskończonymi skokami prędkości stawów w okolicach osobliwości (singularity) np. przy pełnym wyprostowaniu ramienia, klasyczna inwersja Jakobianu ($J^{-1}$) została zastąpiona odporną pseudo-odwrotnością z algorytmem Levenberga-Marquardta.

---

## 🎛️ Inputs and Outputs

**Wejścia do systemu (Inputs):**
* Zadany wektor Twist (prędkości liniowe i kątowe) z joypada: $V_{pad} = [v_x, v_y, v_z, \omega_x, \omega_y, \omega_z]^T$
* Dyskretny sygnał zmiany trybu pracy (Przycisk: Base Frame / Tool Frame)
* Stan układu robota (pozycja przegubów $q$) do aktualizacji macierzy Jakobianu i kinematyki prostej.

**Wyjścia z systemu (Outputs):**
* Wyliczony wektor bezpiecznych prędkości kątowych na stawy ($\dot{q}$), wysyłany bezpośrednio do `joint_group_velocity_controller` robota.

---

## 🚀 Wymagania i Uruchomienie (Dependencies)
* **OS:** Ubuntu 20.04 / 22.04
* **Middleware:** ROS Noetic / ROS 2 (Humble)
* **Robot API:** Kortex API (dla Kinova Gen3)
* **Math Libraries:** `numpy`, `pinocchio` lub `MoveIt!` Core (do rozwiązywania kinematyki na podstawie pliku URDF).

*(Dodaj tutaj swoje własne instrukcje `roslaunch` lub `ros2 run` w zależności od implementacji)*

---
*Autorzy projektu:* [Twoje Imię i Nazwisko]
