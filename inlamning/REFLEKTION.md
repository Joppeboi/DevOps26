# Individuell reflektion

## Produktens livscykel

Tjänsten började som en kopia av template-repot i M1. Den första tiden handlade mer om arbetssätt än om kod. I M2 skyddade jag main med ett ruleset så att allt måste gå via en pull request, och jag testade att en direkt push faktiskt avvisades. Jag fick också lösa en mergekonflikt genom att mergea main in i min branch, välja vilken rad som skulle gälla och ta bort konfliktmarkörerna. Det visade att PR-flödet inte bara är en formalitet, utan stället där ändringar från olika håll möts och granskas.

I M3 paketerades appen i två containrar. Jag lärde mig hur Docker cachar lager och varför beroenden installeras före koden, att EXPOSE bara är dokumentation och att porten öppnas först med -p, och att en .dockerignore håller skräp som __pycache__ borta från imagen. Med docker compose kunde hela appen startas med ett kommando, och första gången pushade jag images till GHCR för hand.

I M4 hittade jag en bugg i summeringen av tecken som alla befintliga tester hade missat, eftersom ingen testade att ta bort en anteckning. Jag skrev ett test som återskapade felet och fixade koden med en rad. I M5 blev testerna en obligatorisk check, Lint and test backend, så att ingen PR kan mergas om de failar. I M6 tog GitHub Actions över publiceringen av images: varje merge till main bygger och pushar nya images, och paketen gjordes publika så att en server kan hämta dem utan inloggning.

I M7 tog jag appen till molnet helt för hand i Horizon, med nyckel, security group, instans, floating IP och Docker installerat via SSH. I M8 byggde Terraform samma miljö från en fil, och i M9 kopplades allt ihop så att en merge till main deployar automatiskt till VM:en.

Det som förändrades mest i mitt sätt att jobba var att jag gick från att utföra stegen själv till att beskriva dem i kod. I början var det jag som körde docker push och klickade fram servern. I slutet var det repot, workflowsen och Terraform som visste hur allt hänger ihop, och mitt jobb blev att göra en ändring och låta kedjan ta resten.

## Vad går sönder utan automation?

Den del som visar det tydligast för mig är infrastrukturen, alltså att bygga VM:en med Terraform i stället för att klicka fram den.

När jag byggde M7 för hand fanns många ställen att göra fel utan att märka det direkt. En regel kan hamna som Egress i stället för Ingress, fel flavor kan väljas, eller default-gruppen kan ligga kvar och blockera trafiken. Inget av det syns förrän något slutar svara, och då måste man felsöka bakåt. Det enda som beskrev vad jag gjort var mina skärmdumpar.

Rebuild-demot i M8 visade skillnaden. Jag rev VM:en med destroy -target och körde apply igen, och floating IP:n var exakt samma före och efter, 195.148.22.184, eftersom porten och adressen är egna resurser som inte rivs med servern. Hade jag byggt om för hand hade jag troligen fått en ny adress. Då hade nip.io-URL:en ändrats, och i M9 hade både DEPLOY_HOST och DEPLOY_KNOWN_HOSTS slutat stämma, så att deployen fallerat. Ett enda manuellt misstag hade brutit hela kedjan efteråt.

Samma mönster syns i M4. Buggen nådde main för att inget test kontrollerade det fallet, och det var först när testet fanns och CI körde det automatiskt som den typen av fel inte längre kunde slinka igenom. Automation tar inte bort misstag, men den gör att samma misstag inte kan göras två gånger utan att någon märker det.