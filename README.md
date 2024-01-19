# Wstęp

CKE niestety nie udostępnia rozwiązań maturalnych zadań bazodanowych. Zmusza to do robienia sporego researchu przy każdym zadaniu którego nie jesteśmy w stanie zrobić. Postanowiłem więc zebrać swoje obserwacje aby ułatwić naukę innym.

Nie jest to wprowadzenie dla tych którzy nie wiedzą co to SQL i nigdy nie pisali żadnych zapytań. Dalej będę zakładał, że wiesz co to `SELECT`, `UPDATE`, `ALTER`, `JOIN`, `GROUP BY`, `ORDER BY`, `LIMIT`, funkcje agregujące i typy danych. Czasami celowo pomijam szczegóły jeśli uważam że nie są istotne. Zawsze możesz doczytać więcej w [dokumentacji](https://dev.mysql.com/doc/refman/8.3/en/sql-statements.html)

Na maturze w formule 2023 możesz wybrać oprogramowanie:

> OpenOffice/Apache
> OpenOffice w wersji 4.1 lub
> nowszej albo LibreOffice
> w wersji 5.3 lub nowszej
> (w tym: Write, Calc, Base)
> i pakiet XAMPP z: Apache,
> MySQL (MariaDB), PHP,
> phpMyAdmin


które jest dostępne zarówno pod Linuxem jak i Windowsem. Wszystkie zapytnia które zamieszczam są więc napisane w MySQL. Czasami będę też wspominał o programie phpmyadmin, który służy do zażądzania bazą danych MySQL lub MariaDB. Z maturalnej oferty, chyba jest najlepszym wyborem do pisania i wykonywania wszystkich zapytań. Pozwala on też na łatwy i szybki import bazy danych z pliku `csv`. Warto dobrze się z nim zapoznać.

Jeśli masz dockera, to w [repozytorium](https://github.com/JakubZojdzik/Maturalny-MySQL) znajdziesz plik `docker-compose.yml`, który posiada obrazy bazy danych MariaDB z hasłem do roota `root` (oczywiście warto je zmienić jeśli planujesz otwierać porty w publicznych sieciach), oraz phpmyadmin dostępnym pod portem `8080`. Utworzony zostanie również volume dla twojej bazy, więc po zdjęciu obrazu nie musisz obawiać się o utratę danych.

Jeśli znajdziesz błąd, lub zechcesz dopisać coś do dokumentu, utwórz pull request ze swoimi zmianami.

Miłego czytania!

# Motywy

Schematy, które pojawiły się na maturze. Starałem się dopisywać konkretne zadania, w których pojawiało się dane zagadnienie, ale niestety nie robiłem tego od początku, więc nie wszędzie znajduje się taki link.

## Brak klucza głównego

Gdy w tabeli podanej w zadaniu nie mamy pewności czy dana kolumna jest unikalna warto dodać nową kolumnę z id. Można to zrobić kwerendą:

```SQL
ALTER TABLE `myTable` ADD COLUMN `id` INT AUTO_INCREMENT UNIQUE FIRST;
```

Lub wyklikać w phpmyadminie: Dodać kolumnę Type=INT, A_I=true, Index=Primary

## Interpretacja stringa jako daty / czasu

Zdarza się, że tabela podana w zadaniu posiada kolumnę z datą lub godziną. Jeśli format daty nie jest odpowiedni, pole to będzie interpretowane w zapytaniu jako tekst i nie będzie możliwości kożystania z [funkcji czasu](#funkcje-czasu).
Przykład: W zadaniu podano datę w formacie `DD/MM/YYYY`. Aby ją zmienić używamy `UPDATE` oraz funkcji `STR_TO_DATE()`:

```SQL
UPDATE wyniki
SET Data_meczu = STR_TO_DATE(Data_meczu, "%d/%m/%Y")
```

Drugim parametrem funkcji to format.

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

Należy trzymać kciuki że nigdy nie będzie zadania z minutami lub sekundami bez zer wiodących, ponieważ taki format nie jest obsługiwany przez funkcję `STR_TO_DATE`. Jeśli jednak cke stwierdzi, że jest to świetny test sprawdzający umiejętności informatyczne maturzystów, można poradzić sobie w następujący sposób:

```SQL
CONCAT_WS('-', HOUR('01:02:03'), MINUTE('01:02:03'), SECOND('01:02:03'))
```

W dodatku należy zmienić typ kolumny na `DATE` / `DATETIME`. Łatwo to wyklikać w phpmyadminie. Należy uważać na precyzję. DATETIME(0) oznacza precyzję do sekund, dla milisekund będzie DATETIME(6). W phpmyadminie możemy to ustawić w polu `Length/Values`. Kwerenda będzie wyglądać tak:

```SQL
ALTER TABLE tabela MODIFY kolumna DATE;
```

## Wiele zapytań w jednym

Zadania:
- [matura 2019 maj](https://arkusze.pl/matura-informatyka-2019-maj-poziom-rozszerzony) Zadanie 6.5
- [Matura 2022 maj](https://arkusze.pl/matura-informatyka-2022-maj-poziom-rozszerzony/) zadanie 6.4

Zagnieżdżanie w sobie zapytań to bardzo przydatny i rozległy temat. Pozwala to na użycie wyniku jednego zapytania `SELECT` w kolejnym zapytaniu. Podzapytanie może zwracać zarówno skalar (liczbę, napis, datę, ...) jak i jeden rekord lub całą tabelę. Możemy je umieścić w 3 miejscach starszego zapytania:

1. `SELECT`

    ```SQL
    SELECT employee_id, last_name,
    (
        CASE WHEN department_id = (
            SELECT department_id from departments WHERE location_id=2500
        ) THEN 'Canada' ELSE 'USA' END
    )
    location FROM employees;
    ```

2. `FROM` (Podzapytanie musi posiadać alias)

    ```SQL
    SELECT m.id_marki
    FROM marki m JOIN pojazdy p ON m.id_marki = p.id_marki
    JOIN
    (
        SELECT m.id_marki, COUNT(DISTINCT p.typ_pojazdu) ilosc
        FROM marki m JOIN pojazdy p ON m.id_marki = p.id_marki
        GROUP BY m.id_marki
    ) A
    ON A.id_marki = m.id_marki
    WHERE ilosc >= 4
    ```

3. `WHERE`

    ```SQL
    SELECT name, population
    FROM city
    WHERE CountryCode IN
    (
        SELECT code
        FROM country
        WHERE region = 'Caribbean'
    )
    ORDER BY population
    LIMIT 5
    ```

Trzeci przypadek często idzie w parze z operatorami:

- `ANY`

    ```SQL
    SELECT s1 FROM t1 WHERE s1 > ANY (SELECT s1 FROM t2);
    ```

    Wyrażenie jest prawdziwe gdy warunek jest prawdziwy dla conajmniej jednej wartości zwróconej przez podzapytanie.

- `ALL`

    ```SQL
    SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
    ```

    Wyrażenie jest prawdziwe gdy warunek jest prawdziwy dla każdej wartości zwróconej przez podzapytanie.

- `IN`

    ```SQL
    SELECT s1 FROM t1 WHERE s1 IN (SELECT s1 FROM t2);
    ```

    Wyrażenie jest prawdziwe gdy wartość s1 występuje chociaż raz w podzapytaniu. W rzeczywistości jest to alias dla `= ANY`.

- `NOT IN`

    ```SQL
    SELECT s1 FROM t1 WHERE s1 NOT IN (SELECT s1 FROM t2);
    ```

    Wyrażenie jest prawdziwe gdy wartość s1 nie występuje ani razu w podzapytaniu. W rzeczywistości jest to alias dla `!= ALL`.

- `EXISTS`

    ```SQL
    SELECT column1 FROM t1 WHERE EXISTS (SELECT * FROM t2);
    ```

    Wyrażenie jest prawdziwe, gdy podzapytanie zwraca conajmniej jeden rekord

## Zliczanie unikalnych wartości (`DISTINCT`)

Zadania:
- [Matura 2015 maj](https://arkusze.pl/matura-informatyka-2015-maj-poziom-rozszerzony/) zadanie 6.4

Czasami chcemy znaleźć na przykład ilość id, które występują w innej tabeli jako klucz obcy w relacji jeden do wielu. Zwykły `COUNT` policzy każde wystąpienie id, nawet jeśli się powtórzy. Aby uzyskać ilość różnych id, można zastosować `COUNT(DISTINCT ...)`. Weźmy przykładową bazę, która zawiera tabelę z pracownikami oraz tabelę z dziennikiem użycia ekspresu do kawy. Chcemy znaleźć liczbę pracowników, którzy robili sobie dzisiaj kawę. Aby zapobiec wielokrotnemu liczeniu jednego pracownika, użyjemy `DISTINCT`:

```SQL
SELECT COUNT(DISTINCT p.id)
FROM pracownicy p JOIN ekspres e ON p.id = e.id_pracownika
WHERE DAY(e.czas) = DAY(NOW())
```

Słowa kluczowego `DISTINCT` możemy również użyć w odniesieniu do `SELECT` (`SELECT DISTINCT ...`). Zwróci on wtedy tylko parami różne rekordy.

## Wartości indukowane warunkiem

Zadania:
- [Matura 2021 maj](https://arkusze.pl/matura-informatyka-2021-maj-poziom-rozszerzony) zadanie 6.5 podpunkt a
- [Matura 2023 maj](https://arkusze.pl/matura-informatyka-2023-maj-poziom-rozszerzony) zadanie 7.3

Może się zdażyć, że w zależności od jakiejś cechy rekordu musimy wprowadzić jakieś ich rozróżnienie. Używamy wtedy klauzuli `CASE`:

```SQL
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END
```

Lub jeśli uzależniamy wartość `CASE` od jednej wartości można skrócić zapis:

```SQL
CASE case_value
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    WHEN valueN THEN resultN
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


Jeśli jesteśmy pytani nie o zestawienie, a o jedną liczbę, należy skożystać z faktu, że funkcje agregujące bez użycia `GROUP BY` domyślnie traktują cały wynik jako jedną grupę. Jeśli więc napisaliśmy zapytanie które zwraca wiersze, których musimy znać np. ilość, możemy otoczyć je kolejnym zapytaniem z `COUNT`:

```SQL
SELECT COUNT(*)
FROM (
    <nasze zapytanie>
) A
```




## Operacje na grupie jako warunek (HAVING)

Zadania:
- [matura 2017 maj](https://arkusze.pl/matura-informatyka-2017-maj-poziom-rozszerzony/) Zad 5.3

`HAVING` umożliwia stworzenie waruneku, który dotyczy grupy a nie wiersza. Dzięki temu można w nim kożystać z funkcji agregujących. Warto zapamiętać, że prawie zawsze chcemy wtedy kożystac z `GROUP BY`. W przeciwnym wypadku, cała tabela będzie rozważana jako jedna grupa.

Weźmy bazę danych z tabelą uczniów i tabelą klas. Chcemy znaleźć klasy, w których jest ponad 20 pełnoletnich uczniów.

```SQL
SELECT k.nazwa
FROM uczniowie u JOIN klasy k ON u.id = k.id_ucznia
WHERE u.wiek >= 18
GROUP BY d.Nazwa
HAVING COUNT(u.id) > 20
```

Ważne jest, że klauzula `HAVING` jest ograniczona przez klauzulę `WHERE`. To oznacza, że jeśli `WHERE` odfiltrował tylko wiersze w których `u.wiek >= 18`, to `HAVING COUNT(u.id) > 20` policzy jedynie pełnoletnich uczniów w danej grupie, i pokaże tylko te grupy, w której ich ilość przekracza 20.

## Działania na zbiorach

Zadania:
- [Matura 2021 maj](https://arkusze.pl/matura-informatyka-2021-maj-poziom-rozszerzony) zadanie 6.5 podpunkt b
- [Matura 2023 maj](https://arkusze.pl/matura-informatyka-2023-maj-poziom-rozszerzony) zadanie 7.3

Co zrobić gdy chcemy odfiltrować pewne rekordy używając `WHERE`, oraz odfiltrować grupy spełniające jakiś warunek przy pomocy `HAVING`, ale jeszcze przed wyrzuceniem rekordów przez `WHERE`. Czasami wystarczy proste zagnieżdżenie zapytań:

```SQL
SELECT ...
FROM (
    SELECT ...
    FROM ...
    HAVING ...
)
WHERE ...
```

Jednak często nie wygląda to najczytelniej, szczególnie gdy zagnieżdżonych jest więcej zapytań. Wtedy bardzo pomocny bywa operator `INTERSECT`. Zwróci on iloczyn dwóch lub więcej zbiorów (wyników zapytania `SELECT`). Załóżmy, że chcemy wyciągnąć z bazy danych informacje na temat klas w szkole, w których jest powyżej 30 uczniów, i jest conajmniej jeden obcokrajowiec. `INTERSECT` może tutaj pomóc:

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

Gdy używamy `INTERSECT` musimy pamiętać, że **kolejność oraz ilość zwracanych kolumn musi być taka sama we wszystkich zapytaniach**. W przeciwnym wypadku dane zostaną źle zinterpretowane.

`INTERSECT` jest tylko jednym z trzech operatorów zbiorów w MySQLu. Pozostałe dwa używamy w analogiczny sposób. Są to:

- `A UNION B`: Oblicza sumę zbiorów A i B
- `A EXCEPT B`: Oblicza różnicę zbiorów A - B (rekordy obecne w A ale nieobecne w B)


## Zdobycie całego rekordu, który ma największą wartość (jakieś pole) w grupie

Zadania:
- [Matura 2019 maj](https://arkusze.pl/matura-informatyka-2019-maj-poziom-rozszerzony/) Zad 6.2


Załóżmy że interesuje nas zestawienie wieku najstarszej osoby w poszczególnych miastach. Jeśli szukamy tylko wieku, wystarczy:

```SQL
SELECT l.miasto, MAX(l.wiek)
FROM ludzie l
GROUP BY l.miasto
```

Problem zaczyna się, gdy chcemy również zdobyć imię i nazwisko każdej z tych osób. Jest to bardzo popularny problem z dużą ilością rozwiązań. Istnieje nawet osobna kategoria pytań na [stack overflow](https://stackoverflow.com/questions/tagged/greatest-n-per-group) poświęcona właśnie temu zagadnieniu. Mi najbardziej podobają się te 2 podejścia:

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

Obydwa rozwiązania zwrócą tylko jeden rekord, nawet jeśli kilka osób będzie miało ten sam wiek.

# Ważne funkcje

Aby lepiej poradzić sobie w warunkach maturalnych (bez internetu), dobrze jest znać, lub przynajmniej wiedzieć o istnieniu poniższych funkcji.

## Funkcje czasu

Jeśli parementrem funkcji jest `jednostka`, to należy wstawić tam jedną z wartości (SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, YEAR).

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

- `LENGTH(napis)`: Długość napisu w bajtach
- `CONCAT(napis1, napis2, napis3, ...)`: Łączy napisy w jeden
- `UPPER(napis)`, `LOWER(napis)`: Podmienia wszystkie litery na odpowiednio wielkie lub małe. Resztę znaków pozostawia bez zmian
- `SUBSTR(napis, start, dlugosc)`: Zwraca podciąg napisu od pozycji startowej o podanej długości
- `LEFT(napis, dlugosc)`, `RIGHT(napis, dlugosc)`: Zwraca odpowiednio pierwsze lub ostatnie znaki napisu
- `TRIM(napis)`, `LTRIM(napis)`, `RTRIM(napis)`: Usuwa białe znaki z odpowiednio początku i końca, tylko początku lub tylko końca napisu
- `POSITION(podciag IN napis)`: Zwraca pierwszą pozycję, na której znajduje się dany podciąg w napisie, lub 0 jeśli podciąg nie istnieje w napisie (pozycje numeruje od 1)

## Funkcje liczbowe

- `ABS(x)`: Moduł z liczby
- `CEIL(x)`, `FLOOR(x)`: Zaokrągla odpowiednio w górę lub w dół do liczby całkowitej
- `a DIV b`: Dzielenie całkowite
- `GREATES(a, b, c, ...)`, `LEAST(a, b, c, ...)`: Odpowiednio największa lub najmniejsza liczba z podanych
- `LOG2(x)`, `LOG10(x)`: Logarytm odpowiednio dwójkowy lub dziesiętny
- `POW(x, a)`: x podniesiony do a-tej potęgi, nie koniecznie całkowitej
- `ROUND(x, a)`: x zaokrąglony do a miejsc po przecinku

## Inne

- `a.liczba BETWEEN 2000 AND 2012`: Wybrany rok będzie zawierał się pomiędzy 2000 i 2012. Zastępuje zapis z operatorami `<=`, `>=`
- `a.ROK IN (2000, 2006, 2012)`: Wybrany rok będzie jedym z podanych: 2000, 2006 lub 2012. Zastępuje zapis z operatorami `=`
