---
layout: post
title: "Podsumowanie tygodnia: 22-28 lutego 2021"
date: 2021-03-02 08:00:00 +0100
tags: [podsumowanie tygodnia, 2021]
---

Żeby ten rok nie skończył się tak, jak poprzedni, gdzie w okolicach Sylwestra zastanawiałem się „kiedy i gdzie ten rok uciekł?!". W tym roku postanowiłem robić cotygodniowe i comiesięczne podsumowania. O tym co się zmieniło w różnych obszarach mojego życia, jak mi idzie z osiąganiem celów i budowaniem nawyków.

Co prawda, blog miał być programistyczny, ale to jest jednak mój blog, więc dlaczego by nie? No i będzie też o rzeczach programistycznych.

# Rozwój

I znów słabo. Przede wszystkim ze względu na natłok obowiązków w pracy i siedzenie po godzinach. Jak zwykle największe niespodzianki i zmiany wpadają na koniec projektu. Ale prawdopodobnie jeszcze tylko tydzień i nowy projekt.

Na bloga dalej nie było specjalnie czasu. Za to było ciekawiej, jeśli chodzi o projekcik z [Micronautem i mikroserwisami](https://github.com/a-mroz/microservices-example).

Tutaj przede wszystkim zabawa z [Zipkinem](https://zipkin.io/), która pokazała mi, że przykładowe requesty, które tam robiłem, nie do końca były najlepszym pomysłem. To mnie zaprowadziło do większego zrozumienia RxJava i tego, jakie korzyści daje brak blokowania. To jeden z tematów, który traktowałem trochę po macoszemu i muszę się mu dokładniej przyjrzeć. Chciałbym też o tym zrobić mały wpis na bloga. Może uda się za tydzień, bo w najbliższych dniach raczej nie będzie na to czasu. Na razie musi wystarczyć, że kilka kropek się połączyło i kilka klepek w głowie ustawiło się we właściwej pozycji ;-)

Poza tym znalazłem świetne wystąpienie Graema Rochera (mam nadzieję, że dobrze odmieniłem) na Devoxxie - [Micronaut Deep Dive](https://www.youtube.com/watch?v=S5yfTfPeue8). Graeme jest twórcą Micronauta i Grails, więc wie, o czym mówi. Pokazuje zalety Micronauta, to jak działa pod spodem, jak radzą sobie bez refleksji i jakie korzyści to przynosi. Pozwoliło mi lepiej zrozumieć framework coraz bardziej mi się podoba.

Dzięki temu wystąpieniu pobawiłem się trochę [introspekcją beanów](https://docs.micronaut.io/1.2.9/api/io/micronaut/core/beans/BeanIntrospector.html). I empirycznie sprawdziłem, że niestety sporo polega na tym, że coś jest publiczne. Mnie [dotknęło](https://github.com/micronaut-projects/micronaut-openapi/issues/398) przede wszystkim to, że w klasach używanych do serializacji przy użyciu Jacksona muszą być prywatne pola i publiczne gettery. Jeśli ich nie ma, to używana jest refleksja, bez żadnego ostrzeżenia. To wymaganie `public` jest moim zdaniem jednym z minusów Micronauta, przynajmniej na ten moment. Da się z tym żyć, ale liczę na to, że usprawnią to w przyszłości.

Jeśli chodzi o naukę AWS, to dalej jedynie fiszki w Anki. Ale odzyskałem dostęp do A Cloud Guru, więc mam nadzieję przyspieszyć w przyszłym tygodniu.

Z rzeczy związanych w pewien sposób z pracą, kupiłem nastawkę na biurko, która pozwala na pracę stojącą. Za pracą stojącą tęskniłem od początku pandemii, bo do takiego biurka nie miałem dotychczas w domu. Zobaczymy, jak sprawdzi się nastawka.

# Sport

W ubiegłym tygodniu bardzo dobrze. Moje Suunto pokazuje 7 aktywności. A pokazywałoby nawet więcej, gdybym nie zapomniał go naładować... No ale przecież nie ćwiczy się dla zegarka, tylko dla siebie.

Było ciepło, więc można było pobiegać i pojeździć na rowerze. Zrobiłem dwa treningi biegowe (5 km i 5.25 km) i trzy razy jeździłem na rowerze - dwa razy na trening i z powrotem (5 km w jedną stronę) i przejażdżka po wałach nad Odrą (20 km). Do tego dwa treningi siłowe.
Byłby jeden trening więcej, gdyby nie obowiązki domowe. No ale nie można mieć wszystkiego.

Niestety nadchodzący tydzień ma być gorszy, jeśli chodzi o pogodę, a pewnie i jeśli chodzi o sport, bo w końcu to powiązane.

Waga, po dłuższej stagnacji, odrobinę w dół, chociaż dalej nie tyle, ile bym chciał.

# Książki

[„The Renaissance Soul: Life Design for People with Too Many Passions to Pick Just One"](https://www.goodreads.com/book/show/415595.The_Renaissance_Soul) dalej w trakcie. Na razie nie jestem specjalnie zachwycony.

[Wino. Kurs wiedzy](https://www.goodreads.com/book/show/24992423-wino-kurs-wiedzy) na ten moment w zasadzie zawieszony, ale planuję wrócić w najbliższym czasie.

Potrzebowałem, dla odmiany, czegoś lżejszego. Zabrałem się za [„Rzeczy Ulotne"](https://www.goodreads.com/book/show/38926130-rzeczy-ulotne) Gaimana. Muszę przyznać, że zaczyna się świetnie - Sherlock Holmes w świecie Lovecrafta? Oj tak. Szkoda tylko, że to jedno opowiadanie, a nie cała powieść.

Przypominam, że jeśli ktoś ma konto w portalu Goodreads, to tutaj można sprawdzić mój [„Reading Challenge"](https://www.goodreads.com/user_challenges/25743441), w którym średnio zwracam uwagę na liczby, ale takie podsumowanie roku to jednak niezła sprawa.

# Podróże

W tym tygodniu niestety nic z podróżowania. Zamiast tego udało się załatwić kilka domowych spraw, a mój gabinet powoli nabiera kształtów :)

# Podsumowanie

To był wyczerpujący tydzień. Niestety, nie do końca takimi sprawami, jakimi bym chciał. Ale pojawiło się też sporo dobrych rzeczy - dużo sportu i nauki związanej z Micronautem i okolicami (Zipkin, RxJava).
