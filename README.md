# Sprawozdanie 1

#### Dawid Grabek

_Początkowo zrobiłem plik word i postarałem się przekleić zadania do pliku README, jednak nie ma tu wszystkich screenów, głównie znajdują się tutaj rozwiązanie na zadanie 4, z potwierdzeniem wykonania zadania i uzasadnieniem_

**W razie jakichkolwiek wątpliwości polecam spojrzeć na plik rozwiazanie.pdf**

## Wstęp

_Całość zadania ma być zrealizowana w przestrzeni nazw (namespace) o nazwie zad1. Na
wstępie należy utworzyć manifest (plik yaml) deklarujący przestrzeń nazw zad1. Następnie
uruchomić ten obiekt (tą przestrzeń nazw). Następnie należy utworzyć zestaw plików
manifestów (plików yaml) opisujących obiekty środowiska Kubernetes zgodnie z poniższymi
założeniami:_

```
# Tworzenie przestrzeni nazw zad1 do sprawozdania

/d/studia/Semestr_9/chmury
$ cat zad1.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: zad1

/d/studia/Semestr_9/chmury
$ kubectl apply -f zad1.yaml
namespace/zad1 created
```

## Zadanie 4

_Należy utworzyć plik yaml definiujący obiekt HorizontalPodAutoscaler, który pozwoli na
utoskalowanie wdrożenia (Deployment) php_ - _apache z zastosowaniem następujących_
minReplicas: 1
maxReplicas: ???????
targetCPUUtilizationPercentage: 50
_Wartość maxReplicas należy określić samodzielnie, tak by nie przekroczyć parametrów quoty
dla przestrzeni nazw zad5. W sprawozdaniu należy UZASADNIĆ PRZYJĘTY DOBÓR WARTOŚCI._

### Uzasadnienie:

W zadaniu 4 ustaliliłem wartość maxReplicas na 5. Uzasadnienie wynika z ograniczeń
nałożonych przez Resource Quota w przestrzeni nazw zad1, które definiują

- Maksymalna liczba podów: **10**
- Dostępne zasoby CPU: **2000m (2 CPU)**
- Dostępna ilość pamięci RAM: **1.5Gi**

**Obliczenia:**

1. Każdy pod w Deployment ma przypisane:

- CPU
  - limits.cpu = 250m
  - requests.cpu = 150m
- RAM:
  - limits.memory = 250Mi
  - requests.memory = 150Mi.

2. Zasoby wymagane dla 1 poda:

- CPU = 250m
- RAM = 250Mi

3. Maksymalna liczba podów ograniczona zasobami:

- CPU: 2000m / 250m=8
- RAM: 1536Mi / 250Mi~=6

4. Najmniejszy wynik (CPU vs RAM) decyduje: maksymalnie **6 podów**.
5. Biorąc pod uwagę, że w namespace może znajdować się również pod worker, który zajmuje:

- CPU: 200m
- RAM: 200Mi

6. Ograniczenia zasobów dla pozostałych podów:

- CPU: 2000m−200m=1800m
- RAM: 1536Mi−200Mi=1336Mi

7. Po podzieleniu zasobów na pod:

- CPU: 1800m / 250m=7.
- RAM: 1336Mi / 250Mi=5.

Finalna liczba replik: **5** (dla zachowania limitów RAM).

## Zadanie 5

_Udokumentowane w pliku pdf, ale każde utworzenie obiektu bazowało na składni:_

```
$ kubectl apply -f <name>.yaml
```

## Zadanie 6

_Ponownie, bazując na przykładach z instrukcji do lab5 i/lub linku podanego w punkcie 3, należy
uruchomić aplikację generującą obciążenie dla aplikacji php_ - _apache i tym samym inicjalizujące
proces autoskalowania wdrożenia tej aplikacji. Za pomocą samodzielnie dobranych poleceń i
wyniku ich działania proszę potwierdzić dobór parametrów z punktu 4._

Wykres obciążenia CPU
![Wykres obciążenia CPU](zadanie6_1.png)

Weryfikacja autoskalowania
![Weryfikacja autoskalowania](zadanie6_2.png)

Zgodnie z wynikami, HorizontalPodAutoscaler (HPA) działa poprawnie. Oto, co się dzieje:

**Obciążenie CPU** : Widać, że CPU w podach przekracza próg 50%, co skutkuje skalowaniem
aplikacji.

Początkowo liczba replik to 1.

Gdy obciążenie CPU wzrosło do 75%, HPA zwiększyło liczbę replik do 2, a finalnie do 4.

**Skalowanie** : HPA poprawnie reaguje na wzrost obciążenia CPU i skaluje liczbę podów, nie
przekraczając ustawionego limitu maxReplicas (5).

**Podsumowanie**

- HPA działa poprawnie, a liczba replik dynamicznie rośnie w odpowiedzi na obciążenie CPU.
- Wszystko jest skonfigurowane zgodnie z oczekiwaniami, a mechanizm autoskalowania działa efektywnie.
