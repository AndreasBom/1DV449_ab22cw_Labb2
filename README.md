#Säkerhets- och prestandaanalys av applikationen Labby Mezzage    
Andreas Bom, ab22cw    
   
   
###Inledning   
Jag kommer i denna rapport presentera säkerhets- och prestandabrister som webbapplikationen Labby Mezzage har, samt ge förslag på hur detta kan förbättras.   
Hädanefter kommer jag att referera den person eller de personer som utför ett hot mot säkerheten för 'hackare, 'hackaren' eller 'hackarna'.   


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
   



   
   
      
   
   

###Referenser   
[1] "Category:Injection," OWASP, 4 November 2007 [Online] Tillgänglig: https://www.owasp.org/index.php/Category:Injection. [Hämtad: 30 November, 2015].   
   
[2] "SQL Injection," OWASP, 14 Augusti 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection. [Hämtad: 30 November, 2015].   
   
[3] "Guide to SQL Injection" OWASP, 9 Juli 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_SQL_Injection . [Hämtad: 30 November, 2015]. 
   
[4] "Cross-site Scripting (XSS)" OWASP,22 April 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS. [Hämtad: 30 November, 2015].    
   
[4] "Cross-site Scripting (XSS)" OWASP,22 April 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS. [Hämtad: 30 November, 2015].


