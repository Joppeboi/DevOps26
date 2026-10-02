# M4 Bugjakt

## 1. Testet som failar

Testet 'test_character_length' i ' backend/tests/test_bugjakt.py' gör samma sekvens som i felrapporten, lägger till 'milk' och 'bread', tar bort sedan bort 'bread' och hämtar summeringen. Förväntat svar, 1 anteckning och 4 tecken.

![Innan fixad kod](m4-test1.png)

## 2. Varför missades buggen?

Buggen missades för att alla de befintliga testerna gick igenom, den tidigare ofixade koden tog aldrig bort några characters.

## 3. Fixade koden

För att fixa koden så att den faktiskt tar bort characters ur objekt som blir raderade behövdes egentligen bara en linje kod, _total_characters -= len(item.text). Samtidigt som den tar bort objektet tas även längden på strängen i objektet bort.

![Fixad kod](m4-test2.png)

## 4. Vad hade behövts för att den här buggen aldrig skulle ha nått main?

För att buggen aldrig skulle ha nått main skulle det nya testet behövts, det testet skapar, tar bort och sedan kollar resultatet.