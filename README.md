# Grid virtual storage - ha addon repository

This add-on allows you to run the Grid Virtual Storage application as a container in a homeassistant installation. The application is an emulator of a virtual energy warehouse, which is often used in net-billing systems for settling energy introduced into the power grid from photovoltaic installations.

This addition is mainly useful for users of the legacy system used in Poland until 2022. Therefore, the rest of the documentation is in Polish.
## Co to jest?
Ten dodatek pozwala monitorować na bieżąco stan wirutualnego magazunu energii  prosumentów rozliczanych w systemie net-billingowym, tzw "na starych zasadach". Po intalacji w HA automatycznie pojawiają się encje:

* **sensor.grid_virtual_storage_grid_main_storage_main_energy_stored** - ilość energii w magazynie (wartość jest już obniżona o  współczynnik sprawności magazyny, zwykle 0.8)
* **sensor.grid_virtual_storage_main_total_grid_forward_energy** - całkowita ilość energii jaka została pobrana z sieci z powodu zbyt małej ilości  energii w mgazynie
* **sensor.grid_virtual_storage_main_total_grid_reverse_energy** - całkowita ilość energii jaka została bezpowrotnie oddana do sieci - jest to wielkość która reprezentuje część jaką zakład energetyczny zabiera sobie w ramach "opłaty" za wirtualny magazyn czyli *(1 - współczynnik sprawności) x ilość energii eksportowanej*
* **sensor.grid_virtual_storage_main_total_storage_charged_energy** - całkowita suma energii jaka została wprowadzona do magazynu (*współczynnik x energia eksportowana*)
* **sensor.grid_virtual_storage_main_total_storage_used_energy** - całkowita ilość energii jaka została pobrana z magazynu

## Instalacja
Dodaj to repozytorium do listy repozytoriów w sekcji dodatków w Home Assistant, możesz użyć przycisku poniżej. Następnie odśwież listę dostępnych, odszukaj dodatek Grid Virtual Storage i wciśnij przycisk instalacji. Dodatek wymaga brokera MQTT, Możesz użyć na przykład dodatku Mosquitto MQTT.
[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https://github.com/techwolfpl/grid-virtual-storage-ha-addon)

## Konfiguracja
**Uwaga. Przed konfiguracją mqtt, zalecam uruchomienie dodatku i wykonanie wstępnego strojenia. Jak to zrobić znajdziesz w następnym rozdziale.**

System wymaga podania **adresu url**, **użytkownika** i **hasła** do brokera MQTT. Opcje te dostępne są w sekcji konfiguracji  dodatku.
Aby dodatek działał poprawnie należy wskazać mu w konfiguracji dwa topiki MQTT na które publikowane wartości:
* **total forward balanced energy** - wskazujący na całkowitą, najlepiej wektorowo zbilansowaną pomiędzy fazami ilość energii pobranej z sieci

* **total reverse balanced energy** - analogicznie topik na którym publikowana jest całkowita ilość energii oddanej do sieci

Bardzo ważne jest aby były to wartości nieskończenie przyrastające - to znaczy aby nie były resetowane np co miesiąc, ale niezależnie od ilości czasu aby ich wartość tylko rosła (dla eksportu energii system wspiera również wartości ujemne, są automatycznie zamieniane na dodatnie.

## Wstępne strojenie magazynu



Aby uzyskać prawdziwe wskzania konieczne jest ustalenie wartości początkowych na których system zacznie pracować. Aby to zrobić musisz znać trzy wartości na dany moment czasu. Najlepszym okresem jest spisanie wartości z topików na dzień rozliczenia - np 1 stycznia. Następnie po otrzymaniu faktury odczytanie pozostałych do wykorzystania kilowatogodzin. Wartości te wprowadzamy po otwarciu interfejsu dodatku.


W sekcji *Grid properties - main*

* **Real forward energy** -  odczyt z wskazanego topiku dla energii pobranej
* **Real reverse energy** - odczyt z wskazanego topiku dla energii oddanej (wartość bezwzględna)

W sekcji Storages 
* **Energy** - ilość energii w magazynie na dzień rozliczenia (upewnij się ze podajesz wartość, realną, już po pomniejszeniu o współczynnik sprawności, różni dystrybutorzy energii mogą podawać na fakturze tą wartość w różny sposób)
* **Charge Efficiency Factor** - współczynnik sprawności magazynu, zwykle jest to 0.8 dla instalacji poniżej 10 kWp oraz 0.7, dla instalacji 10kWp i większych,

Pozostałe wartości powinny być wyzerowane. Formularz zapisywany jest całościowo i jego zapisywanie w trakcie pracy systemu będzie nadpisywać jego wartości. Możesz go użyć w przypadku gdy podczas użytkowania wystąpią jakieś rozbieżności lub pomyłki
