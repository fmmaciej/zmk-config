# zmk-config

Aktualny stan repo i notatki diagnostyczne dla Corne na `nice_nano_v2`.

## Status

- Repo jest przypięte do `ZMK v0.3`.
- Aktualny build matrix buduje tylko:
  - `corne_left`
  - `corne_right`
- Konfiguracja wróciła możliwie blisko znanego dobrego stanu z commita:
  - `a35867df1c7c0db377cd1a262afe1fd328a88e37`

Ten commit był traktowany jako punkt odniesienia, bo:
- underglow/RGB działały poprawnie,
- klawiatura działała stabilnie,
- wyświetlacz nie działał i to był główny temat dalszych prób.

## Aktualna konfiguracja

### Pinning ZMK

- `config/west.yml`: `revision: v0.3`
- `.github/workflows/build.yml`: workflow przypięty do `v0.3`

### Build

`build.yaml` buduje obecnie:

- `nice_nano_v2 + corne_left`
- `nice_nano_v2 + corne_right`

Nie jest obecnie używany:
- `nice_view`
- `nice_view_adapter`

### Display

Aktualnie display jest wyłączony.

W `config/corne.conf`:

- `CONFIG_ZMK_DISPLAY=n`
- `CONFIG_ZMK_WIDGET_OUTPUT_STATUS=n`
- `CONFIG_ZMK_WIDGET_LAYER_STATUS=n`
- `CONFIG_ZMK_WIDGET_BATTERY_STATUS=n`
- `CONFIG_ZMK_WIDGET_WPM_STATUS=n`

Decyzja była praktyczna:

- po włączeniu OLED klawiatura potrafiła okresowo zamulać i zawieszać się,
- pomagał dopiero reset,
- więc display został ponownie wyłączony, żeby wrócić do stabilnego daily-drivera.

### RGB / underglow

W `config/corne.conf`:

- `CONFIG_WS2812_STRIP=y`
- `CONFIG_ZMK_RGB_UNDERGLOW=y`
- `CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=y`
- `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y`
- `CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_USB=n`

W `config/corne.keymap`:

- nadal używany jest include:
  - `../boards/shields/corne/boards/nice_nano.overlay`
- `chain-length` dla LED strip jest nadpisany na `27`
- akcje RGB są aktywne na warstwie function

## Co już zostało ustalone

### 1. `nice_view` nie jest teraz właściwą ścieżką

Wcześniejsze próby z `nice_view` i `nice_view_adapter` kończyły się konfliktami linkera na `v0.3`.

Powód:
- stockowe widgety ZMK dla display kolidowały z widgetami dostarczanymi przez `nice_view`.

Wniosek:
- jeśli dalej testować display, to na razie jako klasyczny display/OLED dla Corne,
- nie wracać teraz do `nice_view`.

### 2. Priorytetem jest zachowanie działającego RGB

Celem nie jest budowanie nowej konfiguracji od zera, tylko rozszerzenie stanu z `a35867df...`, w którym:

- RGB/underglow działały,
- firmware było używalne,
- brakowało tylko działającego displaya.

### 3. `EXT_POWER` nie zostało wyłączone celowo

`CONFIG_ZMK_EXT_POWER=y` jest zostawione świadomie.

To nie było uznane za błąd samo w sobie. Pomysł wyłączenia `EXT_POWER` był tylko hipotezą diagnostyczną pod kątem stabilności displaya, nie docelowym wymaganiem konfiguracji.

## Aktualny problem

Na ten moment:

- klawiatura działa,
- build przechodził po odejściu od `nice_view`,
- RGB/underglow mają być zachowane,
- próby z OLED powodowały niestabilność firmware,
- klawiatura potrafiła się zawieszać albo mocno zamulać,
- do odzyskania normalnego działania potrzebny był reset.

## Poprzednia hipoteza

Najbardziej prawdopodobny praktyczny problem podczas wcześniejszej próby:

- stan `ext_power` mógł zostać kiedyś zapisany jako `OFF`,
- a ZMK potrafi trzymać ten stan w settings,
- więc samo `CONFIG_ZMK_DISPLAY=y` mogło nie wystarczyć, jeśli zasilanie peryferiów pozostawało logicznie wyłączone.

To dobrze tłumaczy sytuację:

- firmware działało,
- konfiguracja display była włączona,
- a ekran pozostawał pusty lub martwy.

## Kolejny sensowny krok

Jeśli temat OLED wróci, najrozsądniejszy kolejny test to jednorazowa próba diagnostyczna:

1. Tymczasowo przywrócić do keymapy akcje:
   - `&ext_power EP_ON`
   - `&ext_power EP_OFF`
2. Wgrać firmware na obie połówki.
3. Na działającej klawiaturze wykonać:
   - `EP_OFF`
   - `EP_ON`
4. Zresetować obie połówki.
5. Sprawdzić najpierw lewą połówkę, bo to ona jest centralna.

## Dodatkowe uwagi

- Obie połówki są czasem podłączane przez USB-C do komputera, ale to nie zmienia logicznych ról splitu w ZMK.
- `corne_right.conf` nie jest już potrzebny, jeśli display pozostaje wyłączony.
- Jeśli kolejna próba z OLED ma być robiona, najlepiej robić ją małymi krokami i nie ruszać jednocześnie RGB, build matrix i konfiguracji split.

## Stan roboczy

Na moment zapisania tej notatki repo jest czyste (`git status` bez zmian).
