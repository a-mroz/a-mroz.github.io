---
layout: post
title: ""
date: 2020-04-29 18:48:14 +0200
---

Mój ostatnim prywatny projekcik [skracacza linków](https://github.com/a-mroz/link-shortener) był ukierunkowany na doszkolenie się z technologii serverless na AWS.

Samo napisanie lambd to oczywiście nie wszystko - trzeba to jakoś zdepoloyować. Robienie tego ręcznie słabo się skaluje i jest błędogenne. Stawiam więc na automatyzację. I tutaj pojawiło się pytanie, co najlepiej wybrać. Opcji jest co najmniej kilka [Terraform](https://www.terraform.io/) , [Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)(w skrócie SAM), framework [Serverless](https://www.serverless.com/) i pewnie jeszcze sporo innych.

Zależało mi na podszkoleniu się z Lambd, DynamoDB, API Gateway. Nauka frameworka do automatyzacji deploymentu nie była głównym celem, więc wybranie czegoś skomplikowanego pewnie zamieniłoby się w prokrastynację na rzecz nauki tej technologii. Potrzebowałem więc czegoś prostego. Stanęło na frameworku Serverless, chociaż warto wziąć też pod uwagę AWS SAM.

# Serverless

Framework powstał w [2015 roku](https://www.serverless.com/company/overview/), więc już chwilę temu. Według twórców jest najczęstszym sposobem, jakego programiści używają do deploymentu FaaS. Wiadomo, że na przechwałki autorów należy patrzeć z przymrużeniem oka, ale patrząc po tym, jak często akurat akurat ten framework pojawia się przy wyszukiwaniach typu "deploying serverless application", to pewnie coś w tym jest.

Poza tutorialami i instrukcjami, które można wyszperać w Googlu (do których ten artykuł dorzuca małą cegiełkę), sam framework ma dobrze zrobioną (przynajmniej na tyle, na ile jej używałem) [dokumentację](https://www.serverless.com/framework/docs/). Poza nieodzowną dokumentacją, dostarczają też sporo [zasobów](https://www.serverless.com/learn/), typu tutoriale, przykłady i artykuły.

Na niewątpliwy plus zaliczyć trzeba, że działa z wieloma chmurami, nie tylko z AWSem. Przy ewentualnej zmianie chmury nie trzeba uczyć się całego frameworka na nowo. Przynajmniej w teorii.

Sporą zaletą jest też [ekosystem pluginów](https://www.serverless.com/plugins/). Użyteczne dla mnie były [serverless-offline](https://www.serverless.com/plugins/serverless-offline/), który emuluje lambdy i API Gateway lokalnie oraz [serverless-dynamodb-local](https://www.serverless.com/plugins/serverless-dynamodb-local/) dzięki któremu można testować DynamoDB lokalnie. Oczywiście pluginy dostarczane przez społeczność bywają różnej jakości i niestety nie wszystkie mi [zadziałały](https://github.com/jubel-han/serverless-event-body-option/issues/1).

Poza samym frameworkiem, który jest open-source, można założyć [konto PRO](https://www.serverless.com/pro/), które dostarcza dashboard, pozwala na monitoring, CI/CD itp. Produkt występuje zarówno w wersji darmowej ("Free for hackers"!), jak i w [wersjach płatnych](https://www.serverless.com/pricing/) - zarówno dla mniejszych zespołów, jak i w wersji enterprise. Mi wersja darmowa wystarcza nawet z naddatkiem.

# Serverless - jak zacząć?

https://www.serverless.com/framework/docs/providers/aws/guide/intro/

- CLI tool (npm, można npx),
