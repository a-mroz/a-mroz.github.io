---
layout: post
title: "Sprawdź swoje zmienne"
date:   2020-05-01 23:21:00 +0200
tags: [linux, zsh, bash, hsql, postgres, env variables]
---

Taka sytuacja: aplikacja w Javie, z użyciem Springa (jeszcze sprzed Spring Boota). Jako baza danych używany PostgreSQL. Do testów integracyjnych używane natomiast HSQLDB, w pamięci.

Connection String do bazy danych, ustawiony w pliku `main/resources/application.properties`, jest zapisany pod propertiesem `database.url`. W testach nadpisany jest w pliku `test/resources/test.properties` i ma wartość `database.url=jdbc\:hsqldb\:mem\:app_db;shutdown=true;hsqldb.write_delay_millis=0`.

Uruchamiam testy, a tam stacktrace długi na jakieś dziesięć ekranów:
```bash
java.lang.IllegalStateException: Failed to load ApplicationContext
        at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:124)
...
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSourceInitializer' defined in class path resource [applicationContext-test.xml]: Invocation of init method failed; nested exception is org.springframework.jdbc.datasource.init.UncategorizedScriptException: Failed to execute database script; nested exception is org.springframework.jdbc.CannotGetJdbcConnectionException:
 Could not get JDBC Connection; nested exception is java.sql.SQLException: Connections could not be acquired from the underlying database!
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1630)
 ...
Caused by: org.springframework.jdbc.datasource.init.UncategorizedScriptException: Failed to execute database script; nested exception is org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get $
DBC Connection; nested exception is java.sql.SQLException: Connections could not be acquired from the underlying database!
...
Caused by: java.sql.SQLException: No suitable driver
        at java.sql/java.sql.DriverManager.getDriver(DriverManager.java:298)

```

I to przy wstawaniu kontekstu Springa, kiedy jeszcze nie za bardzo jest nawet jak podłączyć się debuggerem do swojego kodu aplikacyjnego i sprawdzenia co się dzieje.

# WTF?

{% include image.html
            img="/assets/wtf-1934220_640.jpg"
            alt="WTF"
            url="https://pixabay.com/users/Wokandapix-614097/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1934220"
%}

Pół dnia debuggowania, jak nie dłużej. No bo jak to nie ma drivera? Wcześniej działało... Może winą jest migracja do Javy 11? Może jakiś jar jest uszkodzony? Może... ?

W końcu znajduję problem.

Poprzedni projekt, w którym pracowałem, działał na Heroku, które automatycznie ustawia zmienną `DATABASE_URL` na wartość connection stringa do głównej bazy danych. Do lokalnego uruchamiania aplikacji ustawialiśmy tą zmienną żeby wskazywała na lokalną instancję Postgresa. U mnie w .zshrc.

Niestety, po zamknięciu projektu zapomniałem o tym i zostawiłem wszystko tak, jak było.

Spring, przy rozwiązywaniu wartości propertiesów, kieruje się hierarchią - zmienne środowiskowe mają priorytet przed wartościami w aplikacyjnych plikach properties. A ponieważ nazwy zmiennych środowiskowych zazwyczaj nie mogą mieć w sobie kropek i pisane są wielkimi literami, to properties database.url odpowiada zmiennej środowiskowej `DATABASE_URL`.

Ostatecznie, zamiast łączyć się do bazy HSQLDB, Spring próbował połączyć się do Postgresa używanego w zupełnie innej aplikacji.

Rozwiązanie? `unset DATABASE_URL` lub usunięcie tej zmiennej z pliku konfiguracyjnego powłoki.

Tylko tyle i aż tyle.

# Podsumowanie
Mam nadzieję, że wyżej opisana sytuacja pokazuje jak duży wpływ na naszą aplikację ma ustawienie zmiennych środowiskowych.

O ile w tym przypadku nie był to jakiś wielki problem, bo były to różne bazy, miały różne schematy, użytkowników i hasła. Ale co by się stało gdybyśmy, z jakiegoś powodu, mieli ustawioną zmienną z connection stringinem do produkcyjnej bazy danych, a konfiguracja pozwoliłaby na takie połączenie? Dane z testów integracyjnych poszłyby na produkcję...

Kilka wniosków.

Po pierwsze, praktycznie każda aplikacja ma swój zestaw zmiennych środowiskowych, tak jak każda nowa technologia, którą się bawimy, czy też używamy w projekcie. Ustawiając je w plikach konfiguracyjnych na poziomie użytkownika niestety robi się bałagan. Dlatego dobrze jest od czasu do czasu przejrzeć pliki konfiguracyjne środowiska.

Warto też trochę poświęcić trochę czasu i skonfigurować system i projekt tak, żeby zmienne dla konkretnego projektu były tylko używane tylko w tym projekcie, a nie w całym systemie (np. w NodeJS fajnie działa paczka [dotenv](https://www.npmjs.com/package/dotenv)). Pozostałą konfigurację, która nie zależy od projektu, warto wrzucić w swoje repozytorium `dotfiles` na Githubie - to jest mój plan na najbliższy czas.

Po drugie, przypilnować żeby w bazach produkcyjnych nie dało się tak łatwo namieszać. Tutaj może wystarczyć zapytanie się odpowiednich ludzi jaka jest obecna sytuacja i jak można podłączyć się do bazy produkcyjnej.

No i, po trzecie, przy następnym `java.sql.SQLException: No suitable driver`, zamiast tracić pół dnia na szukanie problemów z bazą in-memory najpierw sprawdź czy przypadkiem nie nadpisujesz sobie propertiesów w plikach `.*rc`.