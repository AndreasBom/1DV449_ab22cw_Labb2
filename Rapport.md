#Säkerhets- och prestandaanalys av applikationen Labby Mezzage    
Andreas Bom, ab22cw    
   
   
###Inledning   
Jag kommer i denna rapport presentera säkerhets- och prestandabrister som webbapplikationen Labby Mezzage har, samt ge förslag på hur detta kan förbättras.   
Hädanefter kommer jag att referera den person eller de personer som utför ett hot mot säkerheten för 'hackare, 'hackaren' eller 'hackarna'.   

##Säkerhet   

###SQL injections   
#####Beskrivning   
Injections är en säkerhetsbrist som ingår i kategorin 'Attack'. SQL injections är en av flera olika typer av injections som kan utföras[1]. Förfarandet går till genom att hackaren injecerar, eller infogar, en SQL-fråga i ett indatafält i klienten, som sedan exekveras på servern mot databasen. Vid en lyckad attack kan hackaren komma åt data från databasen, göra förändringar i databasen eller utföra operationer som är avsedd enbart för databasadministratören[2].   
#####Orsak    
SQL injections kan utföras om data tillåts komma från en opålitlig källa, eller om indata dynamiskt utgör SQL-frågan på servern[2].   

#####Förebyggande åtgärder   
Det finns flera åtgärder för att undvika SQL-injections[2].    
1) Parameteriserade SQL-frågor   
2) Användning av lagrade procedurer   
3) Sanera input från användaren, så att inga 'farliga' tecken släpps fram till SQL-frågan.   
   
Rekommendationen är att i första hand använda parameteriserade SQL-frågor, därefter lagrade procedurer. Att använda sig av åtgärden sanera input från användaren medför större risker än de föregående två alternativen och ska användas med stor försiktighet[2].   
Utöver de ovan nämda åtgärderna rekommenderas att privilegier till användare har så låga rättigheter som är möjligt. D.v.s låt inte användare som enbart ska läsa eller skriva till databasen, ha rättigheter att kunna förändra databasstrukturen(radera databas, skapa nya användare etc.). Rekommendationen är också att använda sig av validering av indata så att felaktig eller skadlig indata aldrig når SQL-frågan[2].

####Labby Mezzage 
Applikationen har en inloggningssida där användaren måste ange emailadress och tillhörande lösenord för att kunna logga in och ta del av meddelanden.   
   
#####Problem   
Det är möjligt att logga in på applikationen utan att känna till email eller lösenord. Inloggning kan ske genom att skriva in en giltig emailadress (viken som helst) och istället för lösenord skrivs 1'or'1'='1 . Det som sker är att en SQL injeceras i lösenordsfältet och oavsett emailadress eller lösenord behöver vara korrekta, eftersom det sista som SQL-frågan säger är att 1=1. Detta kommer alltid vara sant, och innebär att hackaren loggas in.    
   
#####Förslag på lösning   
OWASPs rekommendation för att minimera risker med SQL injections är att implementera antigen parameteriserade SQL-frågor eller parameteriserade lagrade procedurer. Utöver detta bör previligerierna för databasen ses över, för att säkerställa att användarna har minsta möjliga rättigheter, utifrån vad de förväntas kunna göra[3].    
   
   
   
   

###XSS (Cross site scripting)    
#####Beskrivning   
XSS är en attack där hackaren injecerar skadligt script i en annars ofarlig och pålitlig webbsida. När andra användares webbläsare läser in scriptet, execveras det utan att veta att det är script som inte är pålitligt. Scriptet kan få access till användarens kakor, sessions tokens eller annan känslig data som lagras i användarens klient, och som sedan kan skickas och användas av hackaren. Syftet kan också vara att förstöra en webbsida genom att scriptet förändrar HTML-koden.[4].  
   
#####Orsak   
XSS möjliggörs genom att data tar sig in i en application genom en opålitlig källa eller att data skickas till en användare utan att ha blivit validerad från skadligt innehåll[4].   
   
#####Förebyggande åtgärder    
OWASP har en rad olika åtgärder som bör implementeras för att minimera risken med XSS[4].   
1)Tillåt opålitlig data endast på tillåtna platser.    
Exempel på platser där opålitlig data kan göra skada är mellan en scripttagg eller styletagg, i en HTML-kommentar, i ett attributnamn eller taggnamn.    
2) Sanera opålitlig data innan det används i
   *HTML element   
   *HTML attribut   
   *Data för Javascript    
   *Json data
   *Styles properties    
   *URL parameter    
Tillåt inte tecken så som <>&'"/ som output till dokumentet, utan använd istället dess HTML-motsvarighet (\&amp;, \&lt;, \&gt; etc. ).   
3)Om HTML ska tillåtas skickas in till applikationen, använd ett bibliotek som är avsedd till att sanera otillåten data.    
   

####Labby Mezzage   
När användaren är inloggad kan meddelanden skickas, vilka sedan visas i en lista.
   
#####Problem    
Applikationen tillåter att 'farliga' tecken skrivs ut i listan med meddelanden. Det innebär att XSS är möjlig. Genom att skicka ett meddelande som t.ex. \<style>div{ background-color:black;}\</style> förändras sidans utseende och samtliga div-taggar (om de inte ändras  efter meddelandet) kommer att få en svart bakgrund. Att hämta ut värdet på sessionskakan är också fullt möjligt med följande script: \<a href="javascript: alert(document.cookie)">Link to something funny\</a>. Det går t.o.m att rendera upp ett formulär med fält och submit-knapp.    
\<form method="post" action="http://web.andreasbom.se">   
Name:   \<input type="text" name="name">   
Password: \<input type="text" name="Password">   
\<input type="submit" value="Logga in">   
\</form>     
   
#####Förslag på åtgärd   
OWASP rekommenderar att all användardata saneras och encodas till HTML-vänliga tecken innan de renderas till dokumentet. Det innebär att tecken som <>'"&/ aldrig renderas ut, istället kommer det att renderas \&gt;, \&amp; etc.   

   
###Insecure Direct Object References
#####Beskrivning   
Detta är en attack som kan inträffa när ett internt implementerat objekt (ex. en fil, en katalog, data från databas eller en nyckel) refereras genom URL'en eller parameter i ett formulär, och när access till detta objekt kan nås utan auktorisering[5]. Det innebär att obehöriga kan få tillgång till t.ex. en fil eller annan data. 

#####Orsak   
Applikationer som använder namnet på ett internt objekt (kan tex vara id på en användare) i URL'en, eller i formulärparameter, och inte verifierar att användaren har rättigheter att visa aktuell information[5]. 

#####Förebyggande åtgärder
För att undvika att obehöriga får tillgång till data, ska programmeraren undvika att göra en direkt refererens till private objekt(ex. undivik att ha primärnyckeln som en del i URL'en). Om direktreferenser används ska en kontroll göras innan datan hämtas, för att kontrollera om användaren har rättigheter att hämta ut datan[5-6]. 

   
   
####Labby Mezzage   
1) De sparade meddelanden skickas som json-data, och finns på url'en /message/data.     
2) Radera meddelanden görs via POST med /delete i headern och messageID i body'n. Det är tänkt att enbard admin ska ha rättigheter att radera. 

####Problem   
1) Användaren behöver inte vara inloggad för att få access till meddelanden. Detta är ett stort säkerhetsproblem, eftersom obehöriga har  tillgång till datan.    
2) Meddelanden kan raderas genom att manipulera en POST-förfrågan till servern. Detta innebär att obehöriga kan radera meddelanden. 
  
####Förslag på åtgärd     
Eftersom det är en direktreferens till datakällan (/message/data och /delete) är rekommendationen att en autensiering görs innan datan visas eller raderas. Endast behöriga (inloggade) ska ha rätt att se jsonfilen och enbart admin ska kunna radera.    
    
       
       
###Prestanda   
För att öka prestandan finns en rad olika åtgärder som utvecklaren kan göra. Enligt Sounders[7] är tidsåtgången för att hämta HTML-dokumentet ca 10-20%. De kvarvarande 80-90% av tiden är den del där utvecklaren kan göra prestandaförbättringar. Nedan redovisar jag några prestandaåtgärder som Labby Mezzage bör överväga att använda sig av. Hädanefter är 'resurser' stipulativt med filer som laddas in i html-dokumentet, d.v.s bilder, javascript-filer, stylesheets, Flash etc.     
    
######Minimera antalet HTML förfrågningar    
Varje resurs som ska laddas in skickas som en HTTP förfrågan, vilket ökar tidsåtgången som en sida behöver för att laddas klart. Prestandan kan förbättras genom att minska antalet HTTP förfrågningar[7]. I Labby Mezzage finns det tre länkar som inte hittas (materialize.min.css, materialize.js och ie10-viewport-bug-workaround.js). Om detta är "gammal kod" som hänger kvar från något som inte används ska inlänkningarna tas bort. Ska de användas måste rätt url specificeras. I dagsläget skickas det onödig HTTP förfrågningar efter dessa, och HTTP-responsen är 404, d.v.s onödig tid ägnas åt något som inte hittas. Taggen <html> har en bild (b.jpg) som inte visas, eftersom anndra taggar högre upp i dokumentet har attribut som skymmer bilden. Ta bort inlänkningen till denna bild. Bilder som finns på sidan (ex logo.jpg) kan göras som en del i HTML-dokumentet (inline images) och hämtas i samma HTTP-förfrågan som själva dokumentet. Formatet är 'data:<mediatype>;<base64><data>. För att göra bilden till base64-data, finns färdiga moduler/metoder, vilket kan läsas om i aktuellt programmeringsspråks dokumentation[7].   
   
######Användandet  av CDN (Content Delivery Network)    
Att lägga sina resurser på en CDN service kan hjälpa till att öka prestandan genom att det kan bli kortare väg från server till client, samt att CDN-servicen kan välja den servern som har minst trafik för tillfället och som har snabbast responstid[7]. Min bedömning är att Labby Mezzage kommer att användas av ett begränsat antal användare och att det vanliga förekommande är att de kommer befinna sig på ungefär samma plats, och därför kommer en CDN-service inte att ha någon avsevärd påverkan på prestandan.   
    
######Cache    
För att undvika att resurser skickas i varje HTTP-respons, bör dessa cachas, d.v.s lagras lokalt hos användaren. För kompabilitet med äldre webbläsare (och proxys) används Expires Headers för att sätta ett 'utgångsdatum'. När denna tidpunkten är passerad kommer responsen att anses vara 'stale', d.v.s förbrukad och resurserna skickas igen. I nyare webbläsare (och proxys) kan Cache-Controle Max-Age användas. Fördelarna med Max-Age är bl.a. att server och client behöver inte ha samma världstid, utan istället räknas antal sekunder[7].    
   
######Komprimera filer    
Textfiler (HTML, scripts, stylesheets, XML, Json) som skickas mellan server och klient kan komrimeras med Gzip om de är tyngre än 1-2Kb. Denna åtgärd minskar totala storleken på innehållet i responsen. HTTP-förfrågans har headern 'Acctept-Encoding: gzip' och HTTP responsen har headern 'Content-Encoding: gzip' när kompression används. Konfigurationen för att möjliggöra gzip görs enklast i HTTP-serverns konfigurationsfil[7].    
    
######Placera stylesheets i början, script i slutet och använd externa filer     
Inline css och javascript, i motsatts till externa filer, är snabbare första gången sidan laddas. Anledningen är att det krävs färre HTTP-requests. Men eftersom eftersom inline-koden är en del av HTML dokumentet, kommer denna data att behöva laddas varje gång dokumentet laddas. När dessa flyttas ut till externa filer minska filstorleken på HTML dokumentet och om cache används för de externa filerna kommer reponstiden (antalet requests) att minska efter första laddningen av sidan[7].     
   
Genom att placera CSS i början av HTML-dokumentet, kan element börja visas allt eftersom de har laddats in i klienten. Detta är har inte särskilt stor påverkan på den faktiska tiden för sidan att laddas in, utan är i stället ett sätt att visa för användaren att webbsidan är på gång, och är snart färdig (progressiv rendering). Alternativet är att en vit bakgrund visas tills att alla element har mottagits från servern och därefter visa sidan i sin hehet. Detta skulle upplevas som att sidan tar längre tid att laddas[7].   
   
Javascript-filer ska om möjligt placeras i slutet på dokumentet. Hela javascript-filen måste läsas in, innan fortsatt fortsatt inläsning av sidan kan göras. När skriptfilen läggs i slutet kan sidan renderas progresivt som tidigare beskrivet. Läggs scriptfilen i början av dokumentet, kommer sidan att upplevas ta längre tid, eftersom det tar längre tid innan element börjar renderas ut[7].   
   
Labby Mezzage har inline CSS som är placerad efter body-tagen. Applikationen har javaskriptsfiler som läses in i head-taggen. Min rekommendation är att detta ändras, så att ovan nämda regler följs.    
   
######Minifiera Javascript   
Genom att ta bort tecken som inte är nödvändinga (minifiering), kan filstorleken reduceras. Mellanslag, radbrytning, tabb och kommentarer är exempel på tecken och tas bort.   
   
######





    




   
   
      
   
   
   
   

###Referenser   
[1] "Category:Injection," OWASP, 4 November 2007 [Online] Tillgänglig: https://www.owasp.org/index.php/Category:Injection. [Hämtad: 30 November, 2015].   
   
[2] "SQL Injection," OWASP, 14 Augusti 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection. [Hämtad: 30 November, 2015].   
   
[3] "Guide to SQL Injection" OWASP, 9 Juli 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_SQL_Injection . [Hämtad: 30 November, 2015]. 
   
[4] "Cross-site Scripting (XSS)" OWASP,22 April 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS. [Hämtad: 30 November, 2015].    
   
[5] "Top 10 2007-Insecure Direct Object Reference" OWASP,18 April 2010 [Online] Tillgänglig: https://www.owasp.org/index.php/https://www.owasp.org/index.php/Top_10_2007-Insecure_Direct_Object_Reference. [Hämtad: 30 November, 2015].
   
[6] "Top 10 2013-A4-Insecure Direct Object References" OWASP, 14 Juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A4-Insecure_Direct_Object_References. [Hämtad: 1 December, 2015].    
    
[7] S. Sounders, "High Performance Web Sites" Sebastopol: O’Reilly Media, 2007.

