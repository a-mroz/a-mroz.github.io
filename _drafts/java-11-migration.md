Ostatnio pomagałem, z doskoku, w jednej aplikacji zmigrować się z Javy 8 do Javy 11. Aplikacja legacy, dość

Java 8 to taki nowy Cobol (link).

Kilka punktów, które mogą pomóc mnie albo komuś innemu w przyszłości:
- Zostaw jdk 8 na dysku, przynajmniej na czas migracji. Czasem jest potrzeba przełączenia się do starszej wersji i sprawdzenia jak coś działało. Do zarządzania wieloma wersjami jdk (i nie tylko) na jednej maszynie polecam [SdkMan](https://sdkman.io/).
- Najpierw sprawdź, czy projekt buduje się na twojej maszynie przy użyciu jdk 8 i czy działają testy. Niby trywialne, ale ja popełniłem błąd i założyłem, że chociaż nie robiłem nic w tym projekcie od dłuższego czasu, to skoro ostatnim razem działało, to teraz też działa. No i najpierw przełączyłem się na jdk 11 i napotkałem na [failujące testy]({% post_url 2020-05-01-sprawdz-swoje-zmienne %}). Gdybym sprawdził to wcześniej, to odpadłaby mi jedna rzecz do sprawdzenia - czy to nie migracja spowodowała problem.

- Przed przełączeniem na nowe JDK lepiej najpierw podnieść wersje bibliotek, tam gdzie to możliwe. Uwaga na zmiany nazw artefaktów, plugin versions do mavena może nie znaleźć nowszych wersji - na przykład...

Jeśli, tak jak przy tym projekcie, przechodzisz z Mockito 1.x na nowsze, to spodziewaj się failujących testów (i dobrze!). Na przykład, w nowszych wersjach Mockito wykrywa niepotrzebne mockowanie, rzucanie checked Exception niezgodnych z sygnaturami metody itd. Więcej informacji https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2 i https://asolntsev.github.io/en/2016/10/11/mockito-2.1/

Przy okazji przejrzyj zależności. W moim przypadku okazało się, że część jest zaszłościami historycznymi i jest już niepotrzebna albo da się ich pozbyć przez modyfikację kilku linijek. Mniej zależności to mniejszy narzut na utrzymanie projektu, zwłaszcza projektach medycznych, gdzie trzeba przygotowywać np. kwartalne raporty z analizami nowych wersji zależności.

- Przełącz się na jdk i zbuduj projekt.

- Jeśli używasz funkcjonalności z JEE, np. JAXB (np. używany przez Hibernate), annotacji `@PostConstruct`, to konieczne będzie dodanie nowych zależności. Java 9 wyrzuciła je ze standardowego JDK.

- Jeśli w projekcie formatowanie dat jest, stref czasowych itd jest istotne i masz na to testy, to spodziewaj się czerwoności.
Unicode Consortium's Common Locale Data Repository (CLDR)

CLDR było dostępne już w jdk 8, ale domyślnie było wyłączone.
Java 9 wprowadza domyślnie zgodność z https://openjdk.java.net/jeps/252 ... Żeby użyć starszej wersji można użyć `-Djava.locale.providers=COMPAT,SPI`.

- aspectj-maven-plugin.version
- maven-compiler-plugin

- `--illegal-access=permit`