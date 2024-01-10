# Wstęp

Matura z informatyki oraz wiele szkół średnich silnie sugeruje naukę programu Microsoft Access w celu rozwiązywania zadań bazodanowych. Według mne, jest to strata czasu na naukę narzędzia, które nie jest powrzechnie wykożystywane ani przydatne. Znacznie bardziej rozwijająca jest nauka pisania zapytań w SQL. Mierząc się z zadaniami maturalnymi uświadomiłem sobie jak bardzo jeszcze nie znam tego języka. Podczas swoich zmagań z kolejnymi zadaniami do których CKE nie udostępniło rozwiązań a jedynie poprawne odpowiedzi, postanowiłem przyspieszyć proces nauki innym i upożądkowałem swoje problemy w jeden plik.

Pragnę zaznaczyć, że nie jest to wprowadzenie dla tych którzy nie wiedzą co to SQL i nigdy nie pisali żadnych zapytań. Dalej będę zakładał, że wiesz co to `SELECT`, `UPDATE`, `ALTER`, `JOIN`, `GROUP BY`, `ORDER BY`, `LIMIT`, funkcje agregujące, typy danych. Jeśli te pojęcia są dla ciebie obce, zacznij od prostrzych tutoriali.

Na maturze w formule 2023 możesz wybrać oprogramowanie:

```
OpenOffice/Apache
OpenOffice w wersji 4.1 lub
nowszej albo LibreOffice
w wersji 5.3 lub nowszej
(w tym: Write, Calc, Base)
i pakiet XAMPP z: Apache,
MySQL (MariaDB), PHP,
phpMyAdmin
```

które jest dostępne zarówno pod Linuxem jak i Windowsem. Wszystkie kawałki kodu które zamieszczam są więc napisane w MySQL. Czasami będę też wspominał o phpmyadmin, czyli programie do zażądzania bazą danych MySQL lub MariaDB. To własnie w tym programie najłatwiej pisać i wykonywać wszystkie zapytania. Pozwala on też na łatwy i szybki import bazy danych z pliku `csv` jak wygląda to na maturze. Warto dobrze się z nim zapoznać aby uprościć sobie dalszą pracę.

Jeśli jesteś zaznajomiony z dockerem, to w tym repozytorium znajdziesz plik `docker-compose.yml`, który postawi ci bazę danych MariaDB z hasłem do roota `root` (oczywiście warto je zmienić jeśli planujesz otwierać porty w publicznych sieciach), oraz phpmyadminem dostępnym pod portem `8080`. Utworzony zostanie również volume dla twojej bazy, więc po zdjęciu obrazu nie musisz obawiać się o utratę danych.

# Motywy

## Klucze, których klucze obce nie występują w innej tabeli (NOT IN)

Przykład (matura 2022 maj): Uczniowie, których id nie występuje w tabeli ewidencje 4 dnia mesiąca:

```SQL
SELECT u.Imie, u.Nazwisko
FROM uczen u
WHERE u.IdUcznia NOT IN (
    SELECT e.IdUcznia
    FROM ewidencja e
    WHERE DAY(e.Wejscie)=6
)
```

## Interpretacja stringa jako daty / czasu

Przykład: W zadaniu podano datę w formacie `%d/%m/%Y`. Automatycznie jest interpretowana jako string. Aby to zmienić używamy `UPDATE`:

```SQL
UPDATE wyniki
SET Data_meczu = STR_TO_DATE(Data_meczu, "%d/%m/%Y")
```

W dodatku należy zmienić typ wiersza na `DATE` / `DATETIME`. Łatwo to wyklikać w phpmyadminie. Należy uważać na precyzję. DATETIME(0) oznacza precyzję do sekund, dla milisekund będzie DATETIME(6). W phpmyadminie możemy to ustawić w polu `Length/Values`. Kwerenda będzie wyglądać tak:

```SQL
ALTER TABLE tabela MODIFY kolumna DATE;
```

Najczęściej używane symbole przy określaniu formatu daty w zadaniach maturalnych to:
- `%Y`: czterocyfrowy rok, np. `2024`
- `%m`: dwucyfrowy numer miesiąca, np. `06`
- `%d`: dwucyfrowy numer dnia miesiąca, np. `28`

Znacznie rzadziej, ale również przydatne mogą się okazać:
- `%y`: dwucyfrowy rok, np. `24`
- `%c`: numer miesiąca bez zer wiodących, np. `6`
- `%e`: numer dnia miesiąca bez zer wiodących, np. `9`

W przypadku formatu czasu, zazwyczaj możliwe jest użycie
- `%T`: czas w 24-godzinnym formacie, oddzielony dwukropkami, zera wiodące wypełniające do 2 cyfr (hh:mm:ss), np. `04:20:00`

Jeśli format jest inny, trzeba skorzystać z:
- `%H`: dwucyfrowa godzina w 24-godzinnym formacie, np. `14`
- `%h`: dwucyfrowa godzina w 12-godzinnym formacie, np. `02`
- `%i`: dwucyfrowa liczba minut, np. `07`
- `%s`: dwucyfrowa liczba sekund, np. `59`

Lub gdy potrzebujemy liczb bez zer wiodących:
- `%k`: godzina w 24-godzinnym formacie, bez zer wiodących, np. `14`
- `%l`: godzina w 12-godzinnym formacie, bez zer wiodących, np. `02`

Należy trzymać kciuki że nigdy nie będzie zadania z minutami lub sekundami bez zer wiodących, ponieważ taki format nie jest obsługiwany przez funkcje MySQLa. Jeśli jednak cke stwierdzi, że jest to świetny test sprawdzający umiejętności informatyczne maturzystów, można poradzić sobie w następujący sposób:

```SQL
CONCAT_WS('-', HOUR('01:02:03'), MINUTE('01:02:03'), SECOND('01:02:03'))
```

## Operacje na grupie, jako warunek (HAVING)

Zadania:
- [matura 2017 maj](https://arkusze.pl/matura-informatyka-2017-maj-poziom-rozszerzony/) Zad 5.3

Having umożliwia warunek, który dotyczy grupy a nie wiersza. Dzięki temu można w nim kożystać z funkcji agregujących i prawie zawsze chcemy kożystac z `GROUP BY`. W przeciwnym wypadku, cała tabela będzie rozważana jako jedna grupa.

Przykład (matura 2017 maj): tabela `druzyny` zawiera nazwy dróżyn i ich id, tabela `wyniki` bramki stracone i zdobyte w danym meczu przeciwko dróżynie A. Relacja odpowiednio jeden do wielu. Chcemy znaleźć nazwy dróżyn, które mają łączny bilans bramek z dróżyną A równy 0 - tyle samo straconych co zdobytych. Można użyć `HAVING` z funkcją agregującą `SUM`:

```SQL
SELECT d.Nazwa
FROM wyniki w JOIN druzyny d ON w.Id_druzyny=d.Id_druzyny
GROUP BY d.Nazwa
HAVING (SUM(w.Bramki_zdobyte) = SUM(w.Bramki_stracone))
```

Ważne jest, że klauzula `HAVING` jest ograniczona przez klauzulę `WHERE`. To oznacza, że jeśli `WHERE` odfiltrował tylko wiersze w których wiek>=18, to `HAVING COUNT(id_ucznia) > 20` policzy jedynie pełnoletnich uczniów w danej grupie, i pokaże tylko te grupy, w której ich ilość przekracza 20.

## Zdobycie całego rekordu, który ma największą wartość (jakieś pole) w grupie

Zadania:
- [Matura 2019 maj](https://arkusze.pl/matura-informatyka-2019-maj-poziom-rozszerzony/) Zad 6.2

Załóżmy że interesuje nas zestawienie wieku najstarszej osoby w poszczególnych miastach. Jeśli szukamy tylko wieku, wystarczy:

```SQL
SELECT l.miasto, MAX(l.wiek)
FROM ludzie l
GROUP BY l.miasto
```

Problem zaczyna się, gdy chcemy również zdobyć imię i nazwisko każdej z tych osób. Są dwa podejścia:

1. `JOIN` z tabelą wiążącą miasto z wiekiem (widoczną powyżej):
```SQL
SELECT B.miasto, A.imie, A.nazwisko, B.wiek
FROM ludzie A JOIN (
    SELECT l.miasto, MAX(l.wiek) wiek
    FROM ludzie l
    GROUP BY l.miasto
) B ON A.wiek = B.wiek AND A.miasto = B.miasto
GROUP BY B.miasto
```

2. Sprytny `Self JOIN`. Każdy rekord z A łączymy z rekordem z B wtedy i tylko wtedy gdy wiek z B jest większy. Jeśli nie ma rekordu, który ma większy wiek, do każdego pola z B wpisane zostaną NULLe, ponieważ używamy `LEFT JOIN`. Mamy wtedy pewność, że pola z A reprezentują najstarszą osobę:
```SQL
SELECT A.miasto, A.imie, A.nazwisko, A.wiek
FROM ludzie A LEFT JOIN ludzie B
ON A.miasto = B.miasto AND A.wiek < B.wiek
WHERE B.wiek IS NULL
```


Obydwa podejścia zwrócą tylko jeden rekord, nawet jeśli kilka osób będzie miało ten sam wiek.


## Dwa warunki, które ciężko ze sobą pogodzić w jednym zapytaniu

Zadania:
- [Matura 2021 maj](https://arkusze.pl/matura-informatyka-2021-maj-poziom-rozszerzony) zadanie 6.5 podpunkt b
- [Matura 2023 maj](https://arkusze.pl/matura-informatyka-2023-maj-poziom-rozszerzony) zadanie 7.3

Ta sytuacja może mieć miejsce, gdy chcemy odfiltrować pewne rekordy używając `WHERE`, oraz odfiltrować grupy spełniające jakiś warunek przy pomocy `HAVING`, ale jeszcze przed wyrzuceniem rekordów przez `WHERE`. Czasami wystarczy proste zagnieżdżenie zapytań:

```SQL
SELECT ...
FROM (
    SELECT ...
    FROM ...
    HAVING ...
)
WHERE ...
```

Jednak często nie wygląda to najczytelniej, szczególnie gdy zagnieżdżonych jest więcej zapytań. Wtedy bardzo pomocny bywa operator `INTERSECT`. Zwróci on iloczyn dwóch lub więcej zbiorów (SELECTów). Załóżmy, że chcemy wyciągnąć z bazy danych informacje na temat klas w szkole, w których jest powyżej 30 uczniów, i jest conajmniej jeden obcokrajowiec. `INTERSECT` może tutaj pomóc:

```SQL
SELECT k.nazwa
FROM klasa k JOIN uczen u ON u.id = k.id_ucznia
HAVING SUM(u.id_ucznia) > 30
GROUP BY k.nazwa
INTERSECT
SELECT DISTINCT k.nazwa
FROM klasa k JOIN uczen u ON u.id = k.id_ucznia
WHERE u.kraj != "Polska"
```

Pierwsze zapytanie zwróci klasy w których jest powyżej 30 uczniów, drugie zapytanie pokaże klasy w których jest jakiś obcokrajowiec. `INTERSECT` łączy to w jedno zapytanie i pokaże tylko klasy spełniające obydwa te warunki. Warto zaznaczyć, że gdy spróbujemy w najprostrzy i trochę intuicyjny sposób połączyć to w jednego SELECTa, zapytanie będzie **niepoprawne, i może zwrócić błędną odpowiedź**:

```SQL
SELECT DISTINCT k.nazwa
FROM klasa k JOIN uczen u ON u.id = k.id_ucznia
WHERE u.kraj != "Polska"
GROUP BY k.nazwa
HAVING SUM(u.id_ucznia) > 30
```

To zapytanie poda nam klasy, w których jest ponad 30 obcokrajowców, a nie tego oczekiwaliśmy. Na szczęście, sama składnia MySQLa podpowiada nam jak się zachowa, i wymusza umieszczenie `WHERE` nad `GROUP BY` i `HAVING`. W ramach ćwiczenia, warto zastanowić się, jak poprawnie wykonać to zapytanie bez użycia operatora `INTERSECT`.

Gdy używamy `INTERSECT` musimy pamiętać, że **kolejność oraz ilość zwracanych kolumn musi być taka sama we wszystkich SELECTach**. W przeciwnym wypadku dane zostaną źle zinterpretowane.

## Wartości indukowane warunkiem

Warunki tworzymy klauzulą `CASE`:

```SQL
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END
```

`CASE` można użyć jako kolumna w `SELECT`, lub np. w `ORDER BY`:
```SQL
SELECT CustomerName, City, Country
FROM Customers
ORDER BY
(CASE
    WHEN City IS NULL THEN Country
    ELSE City
END);
```

## Ilość wierszy zwróconych przez `SELECT`

Zadania:
- [Matura 2021 maj](https://arkusze.pl/matura-informatyka-2021-maj-poziom-rozszerzony) zadanie 6.5 podpunkt a
- [Matura 2023 maj](https://arkusze.pl/matura-informatyka-2023-maj-poziom-rozszerzony) zadanie 7.3

Jeśli jesteśmy pytani nie o zestawienie, a o jedną liczbę, należy skożystać z faktu, że funkcje agregujące bez użycia `GROUP BY` domyślnie traktują cały wynik jako jedną grupę. Jeśli więc napisaliśmy zapytanie które zwraca wiersze, których musimy znać np. ilość, możemy otoczyć je kolejnym zapytaniem z `COUNT`:

```SQL
SELECT COUNT(*)
FROM (
    <nasze zapytanie>
) A
```


## Brak kluca głównego

Albo kwerendą:

```SQL
ALTER TABLE `myTable` ADD COLUMN `id` INT AUTO_INCREMENT UNIQUE FIRST;
```

Albo wyklikać w phpmyadminie: Dodać kolumnę Type=INT, A_I=true, Index=Primary


# Funkcje do zapamiętania (lub znalezienia w dokumentacji jeśli będziesz miał do niej dostęp na maturze)

## Funkcje czasu

Jeśli parementrem funkcji będzie `jednostka`, to należy wstawić tam jedną z wartości (SECOND, MINUTE, HOUT, DAY, WEEK, MONTH, YEAR).

- `DATEDIFF(data1, data2)`: różnica dni pomiędzy datami, data1 - data2. Ignoruje liczbe godzin, patrzy tylko na różnice dni
- `TIMESTAMPDIFF(jednostka, czas1, czas2)`: Róznica pomiędzy dowolnymi typami czasu, czas2 - czas1. **ZAOKRĄGLA W DÓŁ!** Np. `TIMESTAMPDIFF(day, "2017-06-15 9:35:35", "2017-06-25 09:34:21") = 9`
- `DATE(czas)`: Wyciąga jedynie datę bez czasu
- `YEAR()`, `MONTH()`, `DAY()`, `HOUR()`, `MINUTE()`, `SECOND()`: Wyciąga konkretną część czasu jako liczba
- `DAYOFWEEK(czas)`: Dzień tygodnia. **1-niedziela, 7-sobota**
- `DAYOFYEAR(czas)`: Podaje dzień roku danej daty
- `ADDDATE(data, INTERVAL wartość jednostka)`: Dodaje czas do daty, np. `ADDDATE("2017-06-15 09:34:21", INTERVAL -15 MINUTE);`
- `ADDTIME(czas1, czas2)`: Dodaje czas2 do czas1, czas2 może być ujemny. Np. `ADDTIME("2017-06-15 09:34:21.000001", "2:10:5.000003")`
- [`STR_TO_DATE(string, format)`](#interpretacja-stringa-jako-daty--czasu)

## Funkcje tekstu

- `LENGTH(napis)`: długość napisu w bajtach
- `CONCAT(napis1, napis2, napis3, ...)`: Łączy napisy w jeden
- `UPPER(napis)`, `LOWER(napis)`: Odpowiednio litery na odpowiednio wielkie lub małe. Resztę znaków pozostawia bez zmian
- `SUBSTR(napis, start, dlugosc)`: Zwraca podciąg napisu od pozycji startowej o podanej długości
- `LEFT(napis, dlugosc)`, `RIGHT(napis, dlugosc)`: Zwraca odpowiednio pierwsze lub ostatnie znaki napisu
- `TRIM(napis)`, `LTRIM(napis)`, `RTRIM(napis)`: Usuwa białe znaki z odpowiednio początku i końca, tylko początku lub tylko końca napisu
- `POSITION(podciag IN napis)`: Zwraca pierwszą pozycję, na której znalazł dany podciąg w napisie, lub 0 jeśli nie znalazł


# Zrobione matury
- 2022 maj
- 2021 maj
- 2020 czerwiec
- 2019 maj
- 2018 maj
- 2017 maj

# Trudne zadania warte powtórzenia:
- [2019 maj](https://arkusze.pl/matura-informatyka-2019-maj-poziom-rozszerzony) Zadanie 6.5
- [2023 maj](https://arkusze.pl/matura-informatyka-2023-maj-poziom-rozszerzony) zadanie 7.4

# TODO
- Operacje na stringach (substringi, długość słowa, REPLACE, ...)
- GREATEST(), LEAST(), połączenie MAX z GREATEST
- Dokładne działanie JOINA
- Zaokrąglenia
- Rzutowanie typów
- DISTINCT
- Punktacja zadań
- ANY
- Specjalne podanie błędnej odpowiedzi
- obsługa phpmyadmina
- UNION, UNION ALL, INTERSECT, EXCEPT