#Säkerhets- och prestandaanalys av applikationen Labby Mezzage    
Andreas Bom, ab22cw    
   
   
###Inledning   
Jag kommer i denna rapport presentera säkerhets- och prestandabrister som webbapplikationen Labby Mezzage har, samt ge förslag på hur detta kan förbättras.   
Hädanefter kommer jag att referera den person eller de personer som utför ett hot mot säkerheten för 'hackare, 'hackaren' eller 'hackarna'.   


###SQL injections   
####Beskrivning   
Injections är en säkerhetsbrist som ingår i kategorin 'Attack'. SQL injections är en av flera olika typer av injections som kan utföras[1]. Förfarandet går till genom att hackaren injecerar, eller infogar, en SQL-fråga i ett indatafält i klienten, som sedan exekveras på servern mot databasen. Vid en lyckad attack kan hackaren komma åt data från databasen, göra förändringar i databasen eller utföra operationer som är avsedd enbart för databasadministratören[2].   
####Orsak    
SQL injections kan utföras om data tillåts komma från en opålitlig källa, eller om indata dynamiskt utgör SQL-frågan på servern[2].   

#####Förebyggande åtgärder   
Det finns flera åtgärder för att undvika SQL-injections[2]. 
1) Parameteriserade SQL-frågor   
2) Användning av lagrade procedurer   
3) Sanera input från användaren, så att inga 'farliga' tecken släpps fram till SQL-frågan.   
   
Rekommendationen är att i första hand använda parameteriserade SQL-frågor, därefter lagrade procedurer. Att använda sig av åtgärden sanera input från användaren medför större risker än de föregående två alternativen och ska användas med stor försiktighet[2].   
Utöver de ovan nämda åtgärderna rekommenderas att privilegier till användare har så låga rättigheter som är möjligt. D.v.s låt inte användare som enbart ska läsa eller skriva till databasen, ha rättigheter att kunna förändra databasstrukturen(radera databas, skapa nya användare etc.). Rekommendationen är också att använda sig av validering av indata så att felaktig eller skadlig indata aldrig når SQL-frågan[2].

####Bakgrund    
Applikationen har en inloggningssida där användaren måste ange emailadress och tillhörande lösenord för att kunna logga in och ta del av meddelanden.   
   
####Problem   
   
   
   
   
   
###Referenser   
[1] "Category:Injection," OWASP, 4 November 2007 [Online] Tillgänglig: https://www.owasp.org/index.php/Category:Injection. [Hämtad: 30 November, 2015].   
   
[2] "SQL Injection," OWASP, 14 Augusti 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection. [Hämtad: 30 November, 2015].   




