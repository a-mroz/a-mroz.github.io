---
layout: post
title: "Migracja JDK 8 -> JDK 11"
date: 2020-05-07 07:40:00 +0200
tags: [java]
---

Java 8 ma już swoje lata i jest już niewspierana, chyba że mamy komercyjne wsparcie od Oracle. Niestety wiele aplikacji ciągle z niej korzysta (niektórzy mówią nawet, że to nowy COBOL).

Ostatnio pomagałem, z doskoku, w jednej aplikacji zmigrować się z Javy 8 do Javy 11. Projekt używa mavena, więc skupię się na nim, w gradle pewnie będzie podobnie.

To będzie raczej trochę wskazówek, które mogą pomóc mnie albo komuś innemu w przyszłości, niż kompletny poradnik.

Po więcej szczegółowych informacji polecam [wpis na blogu Benjamina Winterberga](https://winterbe.com/posts/2018/08/29/migrate-maven-projects-to-java-11-jigsaw/), bardziej obszerny poradnik na [blogu CodeFX](https://blog.codefx.org/java/java-11-migration-guide/) oraz poradniki od Oracle: [migracja do JDK 9](https://docs.oracle.com/javase/9/migrate/toc.htm), [do JDK 10](https://docs.oracle.com/javase/10/migrate/toc.htm) i do [JDK 11](https://docs.oracle.com/en/java/javase/11/migrate/index.html).

# Zanim zaczniesz

Zostaw jdk 8 na dysku, przynajmniej na czas migracji. Czasem jest potrzeba przełączenia się do starszej wersji i sprawdzenia jak coś działało. Do zarządzania wieloma wersjami jdk (i nie tylko) na jednej maszynie polecam [SdkMan](https://sdkman.io/).

Najpierw sprawdź, czy projekt buduje się na twojej maszynie przy użyciu jdk 8 i czy działają testy. Niby trywialne, ale ja popełniłem błąd i założyłem, że chociaż nie robiłem nic w tym projekcie od dłuższego czasu, to skoro ostatnim razem działało, to teraz też działa. No i najpierw przełączyłem się na jdk 11 i napotkałem na [failujące testy]({% post_url 2020-05-01-sprawdz-swoje-zmienne %}). Gdybym sprawdził to wcześniej, to odpadłaby mi jedna rzecz do sprawdzenia - czy to nie migracja spowodowała problem.

# Zależności
Przed przełączeniem na nowe JDK lepiej najpierw podnieść wersje bibliotek/frameworków i pluginów mavena, tam gdzie to możliwe.

## Pluginy mavena

Część pluginów mavena w starszych wersjach nie będzie kompatybilna z Javą 11. Przykładowo `maven-compiler-plugin` przed wersją `3.8.0` domyślnie nie obsługuje Javy 11.

Żeby zobaczyć nowsze wersje pluginów w mavenie można użyć polecenia:

`mvn versions:display-plugin-updates`

Tutaj chyba najlepiej użyć wszystkiego co najnowsze.

Niespodziankę sprawił mi plugin mavena do AspectJ. Niestety plugin od MojoHaus nie wspiera jeszcze nowszej Javy. Na szczęście jeden z programistów stworzył forka, którego można użyć. Jeśli w projekcie napotkasz takie zależności, to polecam (ten wątek)[https://github.com/mojohaus/aspectj-maven-plugin/pull/45].

## Biblioteki

Tak jak przy pluginach mavena, część zależności w starszych wersjach nie zadziała z Javą 11.
Żeby zobaczyć nowsze wersje można użyć polecenia

`mvn versions:display-dependency-updates`

Ale tutaj uwaga na zmiany nazw artefaktów, plugin może nie znaleźć nowszych wersji. Na przykład artefakt `hamcrest-all.jar` od jakiegoś czasu nie jest wspierany i (powinno sie używać)[http://hamcrest.org/JavaHamcrest/distributables] artefaktu `hamcrest.jar`.

To jest też dobry moment żeby zerknąć na strony projektowe zależności i sprawdzić release notes.

Polecam też zwrócić uwagę na biblioteki typu `asm`, `byte-buddy`, `cglib`, `javassist`, `cglib` - bardzo prawdopodobne, że będą potrzebowały aktualizacji.

Przy okazji warto przejrzeć zależności w projekcie. W moim przypadku okazało się, że część jest zaszłościami historycznymi i jest już niepotrzebna albo da się ich pozbyć przez modyfikację kilku linijek. Mniej zależności to mniejszy narzut na utrzymanie projektu, zwłaszcza projektach medycznych, gdzie trzeba przygotowywać np. kwartalne raporty z analizami nowych wersji zależności.

### Mockito

Jeśli w projekcie jest jeszcze Mockito w wersji 1.x, to czas przejść do najnowszej wersji. Tym bardziej, że domyślnie Mockito 1.x nie zadziała w Javie 11 i konieczne jest [trochę zmian](https://github.com/mockito/mockito/issues/1419).

Ale przy przechodzeniu na nowsze Mockito spodziewaj się dużo warningów i failujących testów (i dobrze!). Na przykład, w nowszych wersjach Mockito wykrywa niepotrzebne mockowanie, rzucanie checked Exception niezgodnych z sygnaturami metody itp. Ogólnie mnóstwo rzeczy poprawiających testy. Więcej informacji na [wiki Mockito](https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2) i w [tym wpisie](https://asolntsev.github.io/en/2016/10/11/mockito-2.1/).


# Kompilacja i testy

Przełącz się na jdk i zbuduj projekt. Nie najgorszym pomysłem może być zbudowanie projektu na nowym JDK, ale przy zachowaniu kompatybilności ze starszą wersją i dopiero później ustawienie Javy 11.

```xml
<properties>
    <java.version>8</java.version> <!-- zmiana do 11 -->
</properties>
...

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>${maven-compiler-plugin.version}</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
    </configuration>
</plugin>
```

Tutaj już trzeba sprawdzać rzeczy jedna po drugiej - Google Twoim przyjacielem jest. Kilka rzeczy, które można napotkać:

## Zależności do JEE

Jeśli używasz funkcjonalności z JEE, na przykład:
- JAXB (używany m.in. przez Hibernate),
- CORBA,
- JTA,
- javax.annotation (`@PostConstruct`)

to konieczne będzie dodanie nowych zależności. Posprzątano trochę i w Javie 9 zostały oznaczone jako przestarzałe, a w Javie 11 zostały [wyrzucone ze standardowego JDK](https://openjdk.java.net/jeps/320).

Przykładowo, żeby móc używać `javax.annotation` (w tym `@PostConstruct`, którego jest w projekcie sporo):
```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>${javax.annotation-api.version}</version>
</dependency>
```

## Zależności do sun.* i com.sun.*

No i stało się. Chyba od zawsze ostrzegano, że używanie klas w tych pakietach nie jest bezpieczne i ostatecznie [nie ma już do nich dostępu](https://openjdk.java.net/jeps/260). To spowodowało trochę problemów dla różnych frameworków, ale ostatecznie sytuacja została opanowana.

Na szczęście większość rzeczy została udostępniona w API, w lepszych, bardziej bezpiecznych wersjach. Na przykład `sun.misc.BASE64Encoder` może być zastąpiony `java.util.Base64.getEncoder()`.


## Locale

Od Javy 9 [domyślnie włączone](https://openjdk.java.net/jeps/252) jest używanie standardu [Unicode Consortium's Common Locale Data Repository (CLDR)](http://cldr.unicode.org/). CLDR było dostępne już w JDK 8, ale domyślnie było wyłączone.

Jeśli w projekcie (i testach) polegasz na formatowaniu dat, czasu, walut itd. używając formatterów z JDK, to możesz spodziewać się drobnych problemów. Dla przykładu:

```bash
java.lang.AssertionError:
Expected: is "11:55 AM UTC"
     but: was "١١:٥٥ AM UTC"
```

Żeby użyć starszej wersji można użyć propertiesa `-Djava.locale.providers=COMPAT,SPI`, ale oczywiście lepiej poprawić kod tak, żeby było zgodnie ze standardem.


Dodatkowo mogą pojawić się rzeczy typu:

```bash
java.lang.AssertionError:
Expected: is "100 000"
     but: was "100 000"
```

tutaj przy formatowaniu zmieniono spację na [*non-breaking space*](https://en.wikipedia.org/wiki/Non-breaking_space). To bardzo dobra zmiana, bo zapobiega m.in sytuacjom, kiedy symbol waluty jest w jednej linii, a wartość jest w linii poprzedniej. Tutaj przełączenie propertiesa `java.locale.providers` już nie pomoże i trzeba będzie poprawić kod.

## Warning 'An illegal reflective access operation has occurred'

Wraz z nadejściem modułów Jigsaw zwiększyła się enkapsulacja i zwiększyły się obostrzenia co do komunikacji między modułami. Ostrzeżenie `An illegal reflective access operation has occurred` pojawia się kiedy używamy refleksji, żeby dostać się np. do niepublicznych klas/pól.

Jeśli warning jest spowodowany przez nasz kod, to najlepiej to poprawić (tutaj najlepiej doczytać - plan na przyszłość). Gorzej, jeśli powoduje to jakaś biblioteka, z której korzystamy.

Żeby nie blokować migracji autorzy zdecydowali się na warning, zamiast na błąd kompilacji. Żeby je wyłączyć można użyć argumentu `--illegal-access=permit` przy uruchamianiu.

Dla kodu produkcyjnego zostawiłbym to ostrzeżenie, ale myślę, że w testach można je wyłączyć. Dla pluginu Surefire (analogicznie dla Failsafe):

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${surefire.version}</version>
    <configuration>
        <argLine>--illegal-access=permit</argLine>
    </configuration>
</plugin>
```

# Podsumowanie

Jest tego sporo, a i tak to nie wszystkie problemy, które można napotkać przy migracji do nowej wersji Javy. Ale jeśli Twój projekt dalej używa starego JDK, to warto zacząć go migrować, choćby małymi kroczkami.

Powodzenia!