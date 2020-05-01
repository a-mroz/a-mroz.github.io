---
layout: post
title:  "O zmiennych środowiskowych słów kilka"
date:   2020-04-30 21:58:14 +0200
tags: [linux, zsh, bash, ]
---
Zmienne środowiskowe. Temat niby prosty, ale ostatnio trochę musiałem się natłumaczyć i miałem kilka problemów z nimi związanymi. Stąd ten post (i pewnie kilka kolejnych w przyszłości). Nie zamierzam się silić na opisanie wszystkich niuansów i zastosowań, a raczej na pewnego rodzaju wstępie, który może pomóc osobie wchodzącej w temat.

{% include image.html
            img="/assets/shell-5020859_640.jpg"
            alt="Real life shell"
            caption="Real live shell - David Castillo"
            url="https://pixabay.com/pl/users/phydc-2218105/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5020859"
%}

# Co to jest?

Zmienne środowiskowe, to takie zmienne, które są zdefiniowane na poziomie powłoki (shella) systemu operacyjnego. Mówimy tutaj np. o cmd.exe i Powershell dla Windowsa, czy bash i zsh dla Linuksa/Mac OS.

Zmienne, jak to zmienne, mają swoją nazwę i wartość, na przykład `TEST=abc`. Standardowo, w powłokach linuksowych (na innych się nie znam), nazwy zmiennych zapisuje się wielkimi literami, chociaż nie jest to wymagane - `test=123` również zadziała. Wielkość znaków w nazwie ma znaczenie (case-sensitive) - zmienna `TEST` to nie jest ta sama zmienna co `test`.

# Jak sprawdzić wartość?

Żeby sprawdzić jakie zmienne obecnie są dostępne, można je wylistować z poziomu powłoki przez, na przykład przez polecenia `set`, `env`, `printenv` w bashu/zsh lub `SET` w Windowsie. Żeby sprawdzić wartość konkretnej zmiennej środowiskowej można użyć np. polecenia `echo $ZMIENNA` (pamiętając o $ przed nazwą zmiennej) w bash/zsh.

Ważną cechą zmiennych środowiskowych najpopularniejszych powłok jest to, że nowo tworzony proces dziedziczy zmienne środowiskowe swojego rodzica. Czyli, jeśli jakaś zmienna będzie zdefiniowana w powłoce i z tej samej powłoki uruchomimy nasz program, to bedziemy mieli dostęp do tejże zmiennej, o ile jej nie usuniemy/nadpiszemy (przykłady niżej).

# Jak je ustawić?

Poza standardowymi zmiennymi powłoki (np. `PATH` , `PWD`) można tworzyć swoje własne. Zmienne takie można definiować/zmieniać na kilka sposobów. Główne z nich:

- Bezpośrednio przed uruchomieniem programu

    Taka zmienna nie będzie dostępna dla programów innych niż ten, który obecnie uruchomiliśmy.

    ```bash
    ➜  ~ TEST=456 node
    Welcome to Node.js v12.13.0.
    Type ".help" for more information.
    > process.env.TEST
    '456'
    ```

    Przy okazji, można zauważyć, że aby odczytać wartość zmiennej środowiskowej w node.js używa się `process.env.ZMIENNA`.

- W działającej powłoce, dla wszystkich kolejnych procesów

    Przy użyciu słowa kluczowego `export`. Kolejne uruchomione procesy w tej powłoce będą widzieć tą zmienną (dziedziczenie, o którym była wcześniej mowa). Na przykład:

    ```bash
    ➜  ~ export TEST=123
    ➜  ~ jshell
    |  Welcome to JShell -- Version 11.0.6
    |  For an introduction type: /help intro

    jshell> System.getenv("TEST")
    $1 ==> "123"
    ```

    To jest zmienna tymczasowa, po zamknięciu danej powłoki czy też przy uruchomieniu drugiego okienka konsoli nie będzie dostępna.
    Przy okazji, można zauważyć, że żeby odczytać wartość zmiennej środowiskowej można użyć `System.getenv("ZMIENNA")`.

    Tutaj drobna uwaga: otóż nie zadziała rzecz typu: `export TEST=123 node` - zamiast wyeksportować zmienną `TEST` uruchomić proces `node` wykona się jedynie `export` i w tym przypadku nie całości (przynajmniej w bashu/zsh), bo utnie wszystko od spacji i zostanie jedynie `export TEST=123`.


    Jeśli chcemy, żeby zmienna zawierała w sobie spację musimy albo zapisać jej wartość w cudzysłowie

    ```bash
    ➜  ~ export TEST="ala ma kota"
    ➜  ~ echo $TEST
    ala ma kota
    ```

    albo "wyescapować" spację, czyli powiedzieć powłoce, że spacja tutaj jest zwykłym znakiem, a nie znakiem specjalnym. Zazwyczaj takim znakiem jest backslash (`\`).

    ```bash
    ➜  ~ export TEST=hello\ world
    ➜  ~ echo $TEST
    hello world

    ```

- w plikach konfiguracyjnych powłoki

    O plikach konfiguracyjnych (na przykład .bashrc, .zshrc, .zshenv) można napisać cały, długi post, bo zrozumienie kiedy w danej powłoce odczytany jest który plik, nie jest wcale trywialne.

    Przykładowo - jeśli kiedyś instalowałeś JDK lub jakiś inny program i w instrukcji kazano dopisać rzeczy typu: `export JAVA_HOME=/home/user/jdk` `export PATH=$JAVA_HOME/bin:$PATH` w pliku .bashrc, to tworzyłeś nową zmienną o nazwie `JAVA_HOME` i modyfikowałeś zmienną powłoki `PATH`, która jest odpowiedzialna za to, które programy są widoczne w powłoce bez podawania do nich pełnych ścieżek. Dzięki temu można było się odwoływać do programu `javac` bezpośrednio przez wpisanie `javac` w konsoli, zamiast za każdym razem `/home/user/jdk/bin/javac`.

    Po takiej konfiguracji każda *nowo uruchomiona* powłoka będzie widzieć naszą zmienną. Jeśli potrzebujemy przeładować plik konfiguracyjny w obecnie otwartej powłoce możemy wykorzystać polecenie `source`, na przykład `source .zshrc`.


    Przy okazji, obecnie do zarządzania zainstalowanymi JDK i innymi narzędziami, typu Maven/Gradle obecnie polecam [SdkMan](https://sdkman.io/), o którym więcej innym razem.


# Po co mi to?

No dobrze, to teraz dlaczego te zmienne są użyteczne również w kontekście programistycznym, a nie tylko do modyfikowania zachowań powłoki? Otóż pozwalają na redukcję couplingu między konfiguracją aplikacji i samą aplikacją.

Przykładowo, mając zdefiniowaną zmienną `DATABASE_URL` zamiast zapisywać adres na sztywno w źródłach programu (nie mówiąc już o logice typu - "jeśli moja nazwa serwera to XXX, to podłącz mnie do bazy YYY" zaszytej w kodzie), używamy tejże zmiennej do nawiązania połączenia z bazą. Takie podejście pozwala na łatwą konfigurację połączenia do różnych baz danych na różnych środowiskach - inna baza na produkcji, inna na środowisku deweloperskim, inna na środowisku testowym. Poza programistami to bardzo ułatwia życie ludziom od operations, a taka współpraca jest podstawą filozofii Devops.

Inne, popularne zastosowania zmiennych systemowych to:
- konfiguracja połączenia do innych serwisów - adresy, klucze, nazwy użytkowników itd. (aczkolwiek do zarządzania danymi uwierzytelniającymi są lepsze sposoby, np. [Vault](https://www.vaultproject.io/),
- konfiguracji bibliotek i frameworków (np. Hibernate, Spring),
- konfiguracji aplikacji, którą chcemy zmieniać bez ponownej kompilacji i ponownego deploymentu aplikacji (to jest użyteczne np. w testach manualnych).

Czymś, co bardzo fajnie działa w połączeniu ze zmiennymi środowiskowymi są pliki konfiguracyjne (`.properties`, `.yaml` itd.). Przykładowo, w plikach konfiguracyjnych definiujemy sobie wartości domyślne (używane w testach, lokalnym środowisku itd.), a jeśli chcemy taką wartość nadpisać, to robimy to zmienną środowiskową lub przekazując argumenty bezpośrednio do uruchamianego programu. Tego typu konfiguracja jest bardzo rozwinięta w [Spring Boot](https://docs.spring.io/spring-boot/docs/1.2.2.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config) - mamy całą hierarchię co i kiedy jest odczytywane (co może powodować inne problemy, ale o tym następnym razem).


# Podsumowanie
Zmienne środowiskowe, zwłaszcza w połączeniu z plikami konfiguracyjnymi, to bardzo użyteczne narzędzie w rękach programisty. Nie tylko do modyfikacji zachowania systemu operacyjnego, ale też do konfigurowania naszych aplikacji i uniezależnienia ich od środowiska uruchomieniowego.

Kiedy używamy jakiejś zmiennej, to warto udokumentować, chociażby w pliku `README`, że taka zmienna odpowiada za to i za tamto. To pozwoli zaoszczędzić sporo czasu i nerwów komuś, kto będzie chciał uruchomić i konfigurować naszą aplikację.

Trzeba jednak uważać, zachować zdrowy rozsądek i nie przesadzić z wyciągnięciem na zewnątrz aplikacji wszystkiego co się da, na zasadzie "może kiedyś się przyda". Nie chcemy sytuacji, że do włączenia naszej aplikacji potrzebne jest ustawienie stu zmiennych (gdzie część działa tylko wtedy gdy jakaś inna część jest skonfigurowana), zatańczenia tańca słońca i odmówienia trzech zdrowasiek.

Jak zwykle, ważniejsze od tego żeby wiedzieć jak czegoś użyć jest to, kiedy tego użyć.


<style>
.center-image
{
    margin: 0 auto;
    display: block;
}
</style>