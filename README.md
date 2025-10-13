# 🧠 Home Assistant – Valve Temperature Offset Blueprint

Prosty blueprint automatyzacji Home Assistant, który na podstawie pomiaru zewnętrznego czujnika temperatury aktualizuje offset
(czyli kalibrację) w wybranej głowicy termostatycznej (`climate`). Dzięki temu wskazania termostatu lepiej odzwierciedlają
rzeczywistą temperaturę w pomieszczeniu.

## 🚀 Funkcje

- automatyczna kalkulacja wartości offsetu na podstawie różnicy między czujnikiem zewnętrznym i wbudowanym w głowicy,
- zapis obliczonego offsetu do encji `number` (`*_local_temperature_offset*`) powiązanej z urządzeniem,
- prosta blokada czasowa, aby ograniczyć liczbę zapisów (domyślnie co najmniej co 5 minut).

## 📁 Struktura repozytorium

```
blueprints/
└── automation/
    └── valve_temperature_calibration/
        └── auto_offset_calibration.yaml
```

Skopiuj katalog `blueprints/automation` do folderu `config/blueprints/automation` w instalacji Home Assistant, a następnie
ponownie wczytaj blueprints w interfejsie (`Ustawienia` → `Automatyzacje i sceny` → menu z trzema kropkami → `Załaduj ponownie blueprinty`).

## 🔧 Konfiguracja blueprintu

Blueprint wymaga trzech parametrów:

1. **Głowica termostatyczna** – encja `climate`, której offset ma być aktualizowany.
2. **Zewnętrzny czujnik temperatury** – encja `sensor` (najlepiej z klasą urządzenia `temperature`).
3. **Minimalny odstęp pomiędzy aktualizacjami** – wartość w sekundach (domyślnie 300 s).

> **Uwaga:** blueprint automatycznie wyszukuje encję `number` z fragmentem nazwy `local_temperature_offset` powiązaną z wybranym urządzeniem.
> Jeśli Twoje urządzenie używa niestandardowej encji, upewnij się, że znajduje się ona w tym samym urządzeniu co encja `climate`.

## ℹ️ Jak to działa?

Automatyzacja wykonuje proste obliczenie:

```
nowy_offset = temperatura_z_czujnika_zewnętrznego - (temperatura_z_glowicy - aktualny_offset)
```

Otrzymany wynik jest zaokrąglany do pełnych stopni i zapisywany w encji offsetu. Zapis odbywa się nie częściej niż co określony
interwał czasowy, aby ograniczyć liczbę operacji na urządzeniu.

## 🧪 Testowanie

Blueprint został zbudowany na podstawie działającej konfiguracji użytkownika i nie zawiera dodatkowych zabezpieczeń ani
histerezy – jego działanie odpowiada udostępnionym wcześniej automatyzacjom w YAML.
