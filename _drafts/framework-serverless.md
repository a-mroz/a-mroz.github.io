---
layout: post
title: ""
date: 2020-04-29 18:48:14 +0200
---

Mój ostatnim prywatny projekcik [skracacza linków](https://github.com/a-mroz/link-shortener) był ukierunkowany na doszkolenie się z technologii serverless na AWS.

Samo napisanie lambd to oczywiście nie wszystko - trzeba to jakoś zdepoloyować. Robienie tego ręcznie słabo się skaluje i jest błędogenne. Stawiam więc na automatyzację. I tutaj pojawiło się pytanie, co najlepiej wybrać. Opcji jest co najmniej kilka [Terraform](https://www.terraform.io/) , [Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)(w skrócie SAM), framework [Serverless](https://www.serverless.com/) i pewnie jeszcze sporo innych.

Zależało mi na podszkoleniu się z Lambd, DynamoDB, API Gateway. Nauka frameworka do automatyzacji deploymentu nie była głównym celem, więc wybranie czegoś skomplikowanego pewnie zamieniłoby się w prokrastynację na rzecz nauki tej technologii. Potrzebowałem więc czegoś prostego. Stanęło na frameworku Serverless, chociaż warto wziąć też pod uwagę AWS SAM.

# Serverless

- sporo przykładów i artykułów (ten dokłada kolejną, małą cegiełkę)
- Konto i dashboard
- CLI tool (npm, można npx),

- niezła dokumentacja, tutoriale itp. https://www.serverless.com/learn/
- współpracuje z wieloma chmurami
- Rozszerzalny pluginami https://www.serverless.com/plugins/ (chociaż problemy były)
- Jak zacząć,
