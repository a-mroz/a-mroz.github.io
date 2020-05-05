Ostatnio pomagałem, z doskoku, w jednej aplikacji zmigrować się z Javy 8 do Javy 11. Aplikacja legacy, dość... Projekt używa mavena, więc skupię się na nim, w gradle pewnie będzie podobnie.

Java 8 to taki nowy Cobol (link).

Raczej trochę wskazówek, niż kompletny poradnik. Poradnik: https://blog.codefx.org/java/java-11-migration-guide/ https://winterbe.com/posts/2018/08/29/migrate-maven-projects-to-java-11-jigsaw/

Kilka punktów, które mogą pomóc mnie albo komuś innemu w przyszłości.

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

### Mockito

Jeśli w projekcie jest jeszcze Mockito w wersji 1.x, to czas przejść do najnowszej wersji. Tym bardziej, że domyślnie Mockito 1.x nie zadziała w Javie 11 i konieczne jest [trochę zmian](https://github.com/mockito/mockito/issues/1419).

Ale przy przechodzeniu na nowsze Mockito spodziewaj się dużo warningów i failujących testów (i dobrze!). Na przykład, w nowszych wersjach Mockito wykrywa niepotrzebne mockowanie, rzucanie checked Exception niezgodnych z sygnaturami metody itd. Więcej informacji https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2 i https://asolntsev.github.io/en/2016/10/11/mockito-2.1/

Przy okazji przejrzyj zależności. W moim przypadku okazało się, że część jest zaszłościami historycznymi i jest już niepotrzebna albo da się ich pozbyć przez modyfikację kilku linijek. Mniej zależności to mniejszy narzut na utrzymanie projektu, zwłaszcza projektach medycznych, gdzie trzeba przygotowywać np. kwartalne raporty z analizami nowych wersji zależności.

# Kompilacja i testy

Przełącz się na jdk i zbuduj projekt. Pamiętaj o zmianie wersji JDK w plikach mavena/gradla.

Jeśli używasz funkcjonalności z JEE: JAXB (np. używany przez Hibernate), annotacji `@PostConstruct` itd., to konieczne będzie dodanie nowych zależności. Java 9 wyrzuciła je ze standardowego JDK.

+        <dependency>
+            <groupId>javax.annotation</groupId>
+            <artifactId>javax.annotation-api</artifactId>
+            <version>${javax.annotation-api.version}</version>
+        </dependency>



- Jeśli w projekcie formatowanie dat jest, stref czasowych itd jest istotne i masz na to testy, to spodziewaj się czerwoności.
Unicode Consortium's Common Locale Data Repository (CLDR)

CLDR było dostępne już w jdk 8, ale domyślnie było wyłączone.
Java 9 wprowadza domyślnie zgodność z https://openjdk.java.net/jeps/252 ... Żeby użyć starszej wersji można użyć `-Djava.locale.providers=COMPAT,SPI`.

NBSP

- `--illegal-access=permit` w testach - pojawia się kiedy jakaś biblioteka używa `setAccessible(true)`. W testach można sobie na to pozwolić

- Naprawianie błędów kompilacji
sun.* com.sun.*
