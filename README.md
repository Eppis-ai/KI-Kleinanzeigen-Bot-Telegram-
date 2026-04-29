# KI-Kleinanzeigen-Telegram-Bot
Hier findet ihr eine Anleitung, um neue Kleinanzeigeninserate per KI filtern und analysieren zu lassen und bei guten Deals eine Benachrichtigung per Telegram Bot zu schicken.

Als Elektronik interessierter war ich oft auf der Suche nach defekten Geräten, die aber auch lohnenswert zu reparieren sind. Um die Benachrichtigung von Kleinanzeigen nicht selbst alle analysieren zu müssen, wollte ich, dass die KI diese Anzeigen vorher analysiert und mir eine Nachricht schreibt, wenn es einen guten Deal gefunden hat. 
Wie ich das komplett kostenlos umgesetzt habe, möchte ich euch hier Schritt für Schritt aufzeigen. 

## Vorab Info 
Erstellt wenn möglich ein separaten Kleinanzeigen Konto, die Begründung ist in den Hinweisen am Ende des Dokuments zu finden.

## Schritt 1: Accounts / API-Key erstellen
Damit man eine KI-Analyse komplett kostenlos bekommt, habe ich mich für die KI von Google entschieden (Gemini), es besteht auch die Möglichkeit andere generative KIs zu benutzen, aber eventuell sind diese kostenpflichtig. Die weitere Vorgehensweise könnte bei anderen KIs leicht abweichen.

Um Gemini verwenden zu können, wird ein API-Key benötigt, dieser ist unter der Website: https://aistudio.google.com/ zu erstellen.
1. Website aufrufen
2. 'Get API Key' auswählen
3. 'API-Schlüssel erstellen' anklicken
4. Name vergeben z.B Kleinanzeigen_API
5. 'Schlüssel erstellen' anklicken

Des Weiteren wird ein Account bei der Website: https://www.make.com/en benötigt.
1. Account erstellen
2. Account bestätigen
3. Eventuell neue Organisation erstellen
4. 'Scenarios' auswählen
5. Ein neues Szenario hinzufügen

## Schritt 2: Routine erstellen
In diesem Szenario wird nun die Abfolge bestimmt, um von einer Email zu einer Nachricht über Telegram von der KI zu gelangen.

 1. Modul: Webhooks -> custom mailhook
 2. Modul: Text Parser -> Match pattern
 3. Modul: Http -> make a Request
 4. Modul: Text Parser -> HTML to Text
 5. Modul: Google Gemini AI -> Generate a Response
 6. Modul: Telegram Bot -> Send a Text message or reply

## Schritt 3: Parametrieren
Nachdem alle Module hintereinander
angelegt worden sind, müssen nun die Parameter eingestellt werden. Bis auf ein paar Parameter können nun die meisten bereits ausgefüllt werden.

1. Modul: Hinzufügen eines neuen Mailhooks (Name irrelevant). Dies erzeugt eine neue Email Adresse.
2. Modul: Bei dem Pattern sollte folgendes als Filter gesetzt werden: "(https?:\/\/(?:www\.)?kleinanzeigen\.de\/s-anzeige\/[^\s"'>]+)". Damit nur die URL der Anzeige weitergeleitet wird. Sicherheitshalber habe ich die case-sensitive ausgeschaltet. Desweiteren wird im Text der HTML-Content von Modul 1 gesetzt.
3. Modul: URL ist die Ausgabe vom zweiten Modul (dem Match pattern). Method: Get. Authentication: No Authentication
4. Modul: Bei diesem Modul sollte man die Variablen auswählen, die an die KI weitergegeben werden sollte. Ich habe den Preis, den Titel und die Beschreibung ausgewählt.
5. Modul: Hinzufügen einer Connection Mithilfe des API-Keys, den wir in Schritt 1 erhalten haben. AI-Model ist bei mir derzeit auf Gemini-2.5-flash eingerichtet. Hier kann man im Nachhinein ausprobieren was am zuverlässigsten läuft. Unter messages muss ein neues item angelegt werden: Role: User und unter Parts unter message type Text auswählen. Anschließend kann man den Prompt in das Textfeld unter Text eingeben. Hier habt ihr natürlich alle Freiheiten die euch eine KI bietet, um euch aber kleines Beispiel zu liefern, hier den Prompt den ich verwende:

### Beispielprompt:
Du bist ein Experte für das Reselling von gebrauchter Elektronik auf Plattformen wie eBay und Kleinanzeigen. Deine Aufgabe ist es, den potenziellen Profit eines Artikels zu bewerten.Hier sind die Daten des Artikels: {{13.text}}
Deine Analyse-Schritte:Analysiere den Defekt in der Beschreibung. Ist es ein "Easy Fix" (Displaytausch, Akku, Reinigung) oder ein Totalschaden (Wasserschaden, Mainboard, iCloud-Sperre)?Schätze den aktuellen Marktwert des Geräts im funktionstüchtigem Zustand bei Kleinanzeigen. Ziehe vom Marktwert den Preis des Artikels, ca die geschätzten kosten für Ersatzteile und 10€ Versand/Gebühren ab. Gib nur dann ein JA, wenn der geschätzte Profit nach Abzug aller Kosten über 100€ liegt. Antworte ausschließlich in diesem Format:DEAL: [JA / NEIN / VIELLEICHT]PROFIT-CHANCE: [Geschätzter Betrag in €]BEGRÜNDUNG: [Max. 2 Sätze zum Defekt und Risiko]
VORGEHEN: [Erläutere wie man weiter vorgehen sollte]

7. Modul: Hier wird es bei den Parametern etwas kniffliger, denn dazu müssen wir in Telegram ein paar Vorkehrungen treffen. Wir brauchen nämlich ein Bot-Token und die userid. Bevor wir diese holen können wir bereits unter Text das Result der KI als Variable nehmen. Desweiteren als kleinen Tipp habe ich die URL ebenfalls hinterlegt, um nachher über telegram direkt auf die Anzeige zu kommen. 

## Telegram Bot Token erstellen
In Telegram selbst muss der Bot erstellt werden, dazu muss man oben in der Suchleiste nach dem BotFather suchen (achtet darauf, den mit den verifizierten blauen Haken zu benutzen). Mit der Nachricht /newbot wird ein neuer Bot erstellt, anschließend einen Namen für den Bit wählen und danach den Usernamen (kann und sollte auch beides gleich heißen)  Nun kommt als Nachricht der Token zurück, den wir in make.com benötigen, um beim Modul "Telegram Bot" eine neue Connection anzulegen.

## Telegram userid herausfinden
Um die userid zu erhalten muss man lediglich den "userinfobot" in Telegram anschreiben, dieser antworten mit der ID und weiteren Informationen. Nehmt den offiziellen Bot wo ein blaues Zeichen als Präfix hinterlegt ist. Diese userid muss in make.com im Telegram-Bot unter chatid hinterlegt werden.

## Email forwarding
Ab jetzt steht die Routine, damit dies jedoch automatisiert läuft, muss an die Email von Modul 1 (costum mailhook) automatisch eine Email bekommen, damit die Routine angestoßen wird. Dies kann man je nach Emailhost einstellen
Bei Gmail muss man ein Filter erstellen und unter Outlook heißt es beispielsweise Regel.

### Gmail:
Um diesen Filter zu setzen muss man zwangsweise an einen Laptop/PC oder im Handybrowser die Desktopversion verwenden. In den Einstellung kann unter 'Filter und blockierte Adressen' ein neuer Filter erstellt werden. Dort gibt man den Absender von Kleinanzeigen an, der per Email Benachrichtigung die neuste Anzeige schickt (bei mir: noreply@kleinanzeigen.de). Anschließend soll die Email weitergeleitet werden an die Email, die in make.com im ersten Modul generiert wurde, danach soll die Email gelöscht werden, damit das Postfach nicht zugemüllt wird oder man erstellt einen separaten Ordner dafür und setzt diese auf ungelesen und verschiebt die dorthin. Je nach Vorliebe.

### Outlook:
Bitte hier ebenfalls die Dektopversion von Outlook verwenden.t
Unter Datei -> Einstellungen -> Email -> Regeln kann eine neue Regel hinzugefügt werden. Dort einen Namen aussuchen und unter Bedingung: Von auswählen, dann die Email von Kleinanzeigen (bei mir: noreply@kleinanzeigen.de). Zum Schluss als Aktion: Weiterleiten an die Email von make.com (Modul 1) und anschließend löschen bzw. Auf gelesen setzen und in einen speziellen Ordner verschieben. Je nach Vorliebe.

## Telegram Bot registrieren & Suchauftrag erstellen
Damit der Telegram Bot euch anschreiben darf, müsst ihr ihm das erst erlauben, dafür sucht ihr in Telegram nach euren Bot und schickt ihm /start. 
In Kleinanzeigen müsst ihr natürlich einen Suchauftrag erstellen (Suche speichern), damit ihr benachrichtigt werdet. 

## Abschließende Hinweise
- Wenn ihr die Kleinanzeigen App auf dem Handy habt, bekommt ihr Push-Benachrichtigung, anstatt Emails. Hierfür sollte entweder ein separates Konto für dieses Vorgehen erstellt werden oder die App deinstallieren. Eventuell kann man die App nachher ohne Benachrichtigung neu installieren, das habe ich aber nicht getestet.
- In make.com gibt es ein Kontingent von 1000 Credits und 512MB, damit diese auch 30 Tage ausreichen (werden alle 30 Tage aufgestockt), sollte man die Suchaufträge in Kleinanzeigen bereits deutlich einschränken. Für jede Routine werden nämlich ca. 10 Credits benötigt.
- damit ihr Credits spart sollte man unbedingt zwischen dem Modul 6 und Modul 7 einen Filter setzen. Der filtert bei mir alle Deals die auf Nein oder Vielleicht stehen heraus und ich werde nicht für unnötige Deals benachrichtigt. Dafür in make.com einfach auf die Verbindung klicken und einen Namen für den Filter verwenden. Als Condition in der oberen Zeile das Ergebnis der KI hineinsetzten, darunter: Contains (case insensitive) und das Suchkriterium, in meinem Fall, auf "DEAL: JA" setzen.
- ihr könnt die Routine in der Historie quasi debugen, was welches Modul als Eingabe und Ausgabe hat, für das Feintuning ist das essentiell
- zum Testen kann die Routine getestet werden, indem ihr eine Email von Kleinanzeigen manuell an die Email von Modul 1 schickt. Davor muss man im Editmode auf das Play klicken.

## Unterstützung
Falls du es bis hierhin geschafft hast, gratuliere! 
Ich wünsche viel Spaß bei der lebenserleichternden Methode nach Anzeigen zu suchen. 
Diese Anleitung ist komplett kostenlos. 
Wenn Sie dir jedoch Zeit, Geld oder Nervenzusammenbrüche gespart hat, kannst du ein kleines Dankeschön dafür [ bei Paypal hinterlassen ] https://paypal.me/geschenkgeburtstag

Bei weiteren Fragen oder Anmerkungen oder Feedback gerne melden!

## Haftungsausschluss

Diese Anleitung dient ausschließlich zu Informations und Lernzwecken. Alle automatisierten Bewertungen, KI-Analysen und Einschätzungen stellen keine Garantie für tatsächliche Preise, Wertentwicklungen oder erfolgreiche Käufe dar.

Die Nutzung der beschriebenen Konzepte und Methoden erfolgt auf eigenes Risiko. Ich übernehme keine Haftung für finanzielle Verluste, Fehlentscheidungen oder Schäden, die aus der Nutzung der Anleitung entstehen.

Kleinanzeigen, Preise und Märkte ändern sich stetig. Prüfe Angebote stets eigenständig, bevor du Kaufentscheidungen triffst.

Die KI-Analysen müssen stets eigen auf Richtigkeit überprüft werden. Diese Analyse gilt lediglich als Hilfestellung. 
