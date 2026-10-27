[🇬🇧 English version](README.md) | **🇵🇱 Wersja polska**

# Platforma danych dla farmy fotowoltaicznej — Data Lakehouse

Projekt implementacyjny pracy magisterskiej **„Projekt i implementacja platformy danych w architekturze lakehouse na potrzeby wspomagania decyzji dla farmy fotowoltaicznej”** (Miłosz Skulski, SGH, 2026).

## Cel projektu

Platforma integruje pomiary produkcji energii fotowoltaicznej, archiwalne prognozy pogody i ceny energii, a następnie przygotowuje dane do prognozowania, planowania sprzedaży na rynku dnia następnego (DAM), historycznej symulacji pracy magazynu energii i prezentacji decyzji operatorowi. Ważnym założeniem jest **odtwarzanie wyłącznie informacji dostępnych w momencie historycznej decyzji**, bez wykorzystywania późniejszych obserwacji do jej podejmowania.

Jest to **projekt analityczny i historyczny backtest**, a nie system sterowania rzeczywistą farmą lub magazynem energii.

## Dane i zakres

- **PVDAQ / OEDI (NREL)** — pomiary rzeczywistej instalacji PV o mocy 893 kW DC w Kalifornii.
- **Open-Meteo** — archiwalne prognozy pogody.
- **CAISO OASIS** — ceny rynku dnia następnego (DAM) i rynku czasu rzeczywistego (FMM), wykorzystywane zgodnie z rolą w planowaniu i rozliczaniu.
- Okres danych: **lipiec–październik 2024**; główna symulacja: **październik 2024**.
- Parametry magazynu energii są założeniami badawczymi, a nie opisem rzeczywistego urządzenia przyłączonego do instalacji.

## Architektura

Rozwiązanie jest przygotowane dla **Databricks** i wykorzystuje **PySpark, Spark SQL, Delta Lake, Unity Catalog, MLflow** oraz orkiestrację przez **Lakeflow Jobs**.

Przepływ danych:

1. **Landing zone** — pobranie i zachowanie plików wejściowych wraz z informacją o ich pochodzeniu.
2. **Bronze** — odczyt danych źródłowych i zapis ich reprezentacji tabelarycznej.
3. **Silver** — oczyszczanie, kontrola jakości, standaryzacja czasu, jednostek i struktur.
4. **Gold** — integracja źródeł, odtworzenie historycznej dostępności informacji i przygotowanie zbiorów analitycznych.
5. **Analiza i symulacja** — modele prognostyczne, harmonogram DAM, decyzje magazynu i rozliczenie energii.
6. **Panel operatora** — wyniki oraz instrukcje decyzyjne udostępniane przez Databricks SQL.

## Zawartość repozytorium

Notebooki są zapisane jako pliki `.ipynb` w formacie eksportu Databricks (`# Databricks notebook source`).

| Plik | Zadanie |
| --- | --- |
| `00_setup_project.ipynb` | Przygotowanie i kontrola środowiska projektu. |
| `01_ingest_nlr.ipynb` | Pobranie pomiarów instalacji PV. |
| `02_bronze_nlr.ipynb` | Warstwa Bronze dla pomiarów PV. |
| `03_silver_nlr.ipynb` | Oczyszczanie i standaryzacja pomiarów PV. |
| `04_ingest_weather_forecasts.ipynb` | Pozyskanie historycznych prognoz pogody. |
| `05_silver_weather.ipynb` | Przygotowanie danych pogodowych. |
| `06_ingest_caiso_prices.ipynb` | Pozyskanie cen CAISO. |
| `07_bronze_caiso_prices.ipynb` | Warstwa Bronze dla cen energii. |
| `08_silver_caiso_prices.ipynb` | Przygotowanie cen energii do analiz. |
| `09_gold_market_replay.ipynb` | Integracja Gold i historyczne odtworzenie informacji rynkowej. |
| `10_gold_eda_and_time_split.ipynb` | Analiza eksploracyjna i chronologiczny podział danych. |
| `11_baseline_forecasting_models.ipynb` | Budowa i ocena bazowych modeli prognostycznych. |
| `12_day_ahead_schedule.ipynb` | Przygotowanie harmonogramu sprzedaży DAM. |
| `13_battery_dispatch_backtest.ipynb` | Historyczna symulacja ładowania i rozładowania magazynu. |
| `14_business_results.ipynb` | Obliczenie i prezentacja wyników energetycznych i finansowych. |
| `15_decision_support_system.ipynb` | Przygotowanie informacji i rekomendacji dla operatora. |
| `Panel operacyjny farmy fotowoltaicznej.lvdash.json` | Eksport definicji panelu Databricks. |
| `manifest.mf` | Plik manifestu zawarty w eksporcie projektu. |

## Orkiestracja i konfiguracja Databricks

Projekt obejmuje również konfigurację orkiestracji w **Databricks Lakeflow Jobs**. W repozytorium GitHub znajdują się dwa pliki definicji jobów w YAML oraz główny plik `databricks.yaml`:

- **Job 1 — przygotowanie danych (notebooki 00–09):** inicjalizacja środowiska, pobranie danych źródłowych oraz przetwarzanie warstw Bronze, Silver i Gold.
- **Job 2 — analiza i symulacja (notebooki 10–15):** eksploracja danych, podział czasowy, prognozowanie, planowanie sprzedaży DAM, backtest magazynu i przygotowanie wyników dla operatora.
- **`databricks.yaml`:** główny plik konfiguracji Databricks Asset Bundle, wskazujący pliki zasobów YAML.

Definicje YAML dokumentują konfigurację workflow; samo ich umieszczenie w GitHubie nie oznacza automatycznego wdrożenia ani cyklicznego uruchamiania. W projekcie joby były uruchamiane ręcznie. Ścieżki notebooków i ustawienia workspace należy dostosować do konkretnego środowiska Databricks.

## Dashboard i przykładowe wyniki

Repozytorium zawiera plik **`Panel operacyjny farmy fotowoltaicznej.lvdash.json`** — eksport definicji dashboardu Databricks SQL. Dashboard przedstawia m.in. decyzje dotyczące magazynu, przepływy energii, stan naładowania oraz wyniki finansowe symulacji. Definicja JSON nie jest zrzutem ekranu; do jej wyświetlenia jako interaktywnego panelu potrzebny jest Databricks i odpowiednie tabele wynikowe.

Poniżej przedstawiono **oryginalne zrzuty ekranu z dashboardu Databricks**, udostępnione przez autora projektu. Wszystkie dotyczą **30 października 2024 r.** i pokazują wyniki historycznej symulacji w interwałach 15-minutowych.

### 1. Produkcja PV a harmonogram DAM

![Prognozowana i rzeczywista produkcja PV oraz harmonogram DAM, 30 października 2024](assets/pv_production_schedule_oct2024.png)

**Interpretacja:** Wykres porównuje prognozowaną produkcję PV, rzeczywistą produkcję oraz zaplanowaną sprzedaż na rynku dnia następnego (DAM). Rzeczywista generacja zmienia się w ciągu dnia i odbiega od prognozy oraz harmonogramu. Rozbieżności stanowią kontekst dla historycznej symulacji decyzji magazynu; harmonogram nie jest tożsamy z pomiarem rzeczywistej produkcji.

### 2. Polecenia ładowania i rozładowywania magazynu

![Polecenia ładowania i rozładowywania magazynu, 30 października 2024](assets/battery_dispatch_oct2024.png)

**Interpretacja:** Dodatnia rekomendowana moc oznacza ładowanie, ujemna — rozładowywanie, a zero — brak zmiany stanu magazynu. Wykres pokazuje naprzemienne decyzje w godzinach produkcji PV, wynikające z reguł bilansowania harmonogramu i ograniczeń przyjętego modelu magazynu. To wynik backtestu, a nie polecenia wysłane do rzeczywistego urządzenia.

### 3. Przepływy energii w wybranym dniu

![Przepływy energii PV, magazyn i sieć, 30 października 2024](assets/energy_flows_oct2024.png)

**Interpretacja:** Wykres rozróżnia energię z PV skierowaną bezpośrednio do sieci, energię przeznaczoną na ładowanie magazynu oraz energię oddaną z magazynu do sieci. Pokazuje również łączny eksport. Umożliwia sprawdzenie, jak decyzje magazynu zmieniają rozdział energii w poszczególnych 15-minutowych interwałach.

### 4. Panel operatora — decyzje i realizacja

![Panel operatora z decyzjami i przepływami energii, 30 października 2024](assets/operator_panel_oct2024.png)

**Interpretacja:** Tabela prezentuje lokalny czas interwału, decyzję **UTRZYMAJ / ŁADUJ / ROZŁADUJ**, instrukcję dla operatora oraz wielkości produkcji, eksportu, ładowania i rozładowywania. Widoczny fragment pokazuje m.in. ładowanie o 12:45 i rozładowanie o 13:45. Panel łączy wyniki obliczeń z czytelną instrukcją operacyjną; w projekcie ma charakter informacyjny i nie steruje urządzeniami.

*Źródło wszystkich czterech ilustracji: oryginalny dashboard Databricks projektu, zrzuty ekranu udostępnione przez autora (30.10.2024).*

## Uruchomienie

1. Przygotuj workspace **Databricks** z dostępem do wymaganych funkcji Spark/Delta i odpowiednimi uprawnieniami do Unity Catalog, wolumenów oraz tworzenia tabel.
2. Zaimportuj pliki `.ipynb` jako notebooki Databricks. Zachowaj ich nazwy i kolejność.
3. Otwórz `00_setup_project.ipynb` i sprawdź ustawienia katalogu, schematów, ścieżek oraz uprawnienia w swoim środowisku. Skontroluj też konfigurację źródeł zewnętrznych w notebookach pobierających dane.
4. Uruchamiaj notebooki w kolejności **00–15**. Logicznie odpowiada to dwóm etapom: **00–09 — przygotowanie danych**, **10–15 — analiza i symulacja**. Repozytorium zawiera również definicje dwóch jobów w YAML (wymagające zgodnych ścieżek i konfiguracji środowiska).
5. Po zbudowaniu tabel wynikowych zaimportuj definicję panelu z pliku `.lvdash.json` i dostosuj połączenia oraz nazwy obiektów SQL do swojego workspace.
6. Zweryfikuj komunikaty kontroli jakości, kompletność danych, zakresy dat i wyniki symulacji przed interpretacją rezultatów.

**Uwaga:** repozytorium nie jest samodzielną aplikacją uruchamianą poleceniem `python`. Notebooki wymagają skonfigurowanego środowiska Databricks, danych źródłowych i dostępu do odpowiednich usług. Ten opis nie stanowi gwarancji uruchomienia bez dostosowania konfiguracji.

## Wyniki przykładowej symulacji

Dla października 2024 r., w scenariuszu bilansowania harmonogramu sprzedaży energią z PV, w prezentacji pracy podano:

| Wskaźnik | Wynik |
| --- | ---: |
| Produkcja PV | 109,11 MWh |
| Eksport energii do sieci | 109,15 MWh |
| Straty magazynu | ok. 0,36 MWh |
| Przychód operacyjny netto po koszcie degradacji | 4 579,68 USD |
| Wynik po korekcie końcowego zapasu energii | 4 564,92 USD |

Zapas energii w magazynie zmienił się z 0,50 do 0,10 MWh. Wyniki dotyczą przyjętych parametrów i jednego miesiąca. **Nie są analizą opłacalności inwestycji** — nie obejmują m.in. kosztu zakupu magazynu i części opłat.

## Ograniczenia

- Dane mają charakter historyczny; platforma nie steruje urządzeniami ani nie łączy się bezpośrednio z SCADA/IoT.
- Dostępność prognoz odtwarzana jest z przyjętym założeniem opóźnienia publikacji.
- Ceny wybranego punktu agregacyjnego są przybliżeniem cen lokalizacji farmy.
- Rozliczenia nie obejmują wszystkich zasad rzeczywistego rynku.
- Kontrole jakości sprawdzają zgodność danych i obliczeń z przyjętymi regułami, ale nie potwierdzają poprawności każdego pomiaru źródłowego.

## Możliwe kierunki rozwoju

Regularne pobieranie nowych danych, obsługa danych opóźnionych i korekt, aktualizowanie modeli, rozliczenia według odrębnych harmonogramów oraz integracja szczegółowych pomiarów falowników i informacji o błędach urządzeń.

## Materiały źródłowe

Opis opracowano na podstawie dołączonej pracy magisterskiej, prezentacji obrony oraz struktury archiwum z kodem. Pliki YAML zostały dodane później w repozytorium GitHub i nie występowały w dostarczonym archiwum ZIP. Szczegóły algorytmów, założeń, ograniczeń i weryfikacji znajdują się w pracy magisterskiej, szczególnie w rozdziale 3. Wartości liczbowe powyżej pochodzą z prezentacji, a nie z ponownego uruchomienia kodu.
