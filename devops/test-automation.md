---
tags:
  - devops
  - test-automation
---

# test-automation

## [Apache jmeter](../jmeter)

**Quellen**

* (Beispiele für jmeter)[https://github.com/fgiloux/auto-perf-test]
# [JavaScript cypress](../cypress)

## In einer Nussschale

Cypress ist ein Front-End-Testwerkzeug der nächsten Generation, das für das moderne Web entwickelt wurde. Wir sprechen die Hauptprobleme an, mit denen Entwickler und QS-Ingenieure beim Testen moderner Anwendungen konfrontiert sind.

Wir machen es einfach:

* Tests einrichten
* Schreiben Sie Tests
* Tests ausführen
* Debug-Tests

Cypress wird am häufigsten mit Selen verglichen. Cypress unterscheidet sich jedoch grundlegend und architektonisch. Cypress unterliegt nicht denselben Einschränkungen wie Selenium.

**Quelle:**

* [docs.cypress.io](https://docs.cypress.io/guides/overview/why-cypress.html#In-a-nutshell)
* [nightwatchjs](../nightwatchjs)
# Was ist Nachtwache?

Nightwatch.js ist ein automatisiertes Testframework für Webanwendungen und Websites, das in Node.js geschrieben wurde und die W3C WebDriver-API (früher Selenium WebDriver ) verwendet.

Es handelt sich um eine umfassende End-to-End- Testlösung, die das Schreiben automatisierter Tests und die Einrichtung der Continuous Integration vereinfacht . Nightwatch kann auch zum Schreiben von Node.js-Einheiten- und Integrationstests verwendet werden.

Der Name Nightwatch wurde von dem berühmten Gemälde The Night Watch des niederländischen Künstlers Rembrandt van Rijn inspiriert . Das Meisterwerk wird prominent im Rijksmuseum in Amsterdam - Niederlande gezeigt.
# Überblick über WebDriver

WebDriver ist eine Universalbibliothek zur Automatisierung von Webbrowsern. Es wurde im Rahmen des Selenium- Projekts gestartet, einem beliebten und umfassenden Satz von Tools für die Browser-Automatisierung, der ursprünglich für Java geschrieben wurde und jetzt die meisten Programmiersprachen unterstützt.

Nightwatch verwendet die WebDriver-API , um Aufgaben zur Browser-Automatisierung auszuführen, beispielsweise das Öffnen von Fenstern und das Klicken auf Links.

WebDriver ist jetzt eine W3C-Spezifikation, die die Browserautomatisierung standardisieren soll. WebDriver ist eine Fernsteuerungsschnittstelle, die die Introspektion und Steuerung von Benutzeragenten ermöglicht. Es bietet eine Plattform und eine erholsame HTTP-API, mit der Webbrowser ferngesteuert werden können.
# Theorie der Arbeitsweise

Nightwatch kommuniziert über eine unruhige HTTP-API mit einem WebDriver-Server (wie ChromeDriver oder Selenium Server). Das Protokoll wird durch die W3C WebDriver-Spezifikation definiert , die vom JSON Wire- Protokoll abgeleitet ist. Nachfolgend finden Sie einen Beispielworkflow für die Browserinitialisierung.

In den meisten Fällen muss Nightwatch mindestens zwei Anforderungen an den WebDriver-Server senden, um einen Befehl oder eine Assertion ausführen zu können. Die erste Anforderung besteht darin, ein Element mit einem CSS-Selektor (oder einem Xpath-Ausdruck) zu suchen und das nächste auszuführen der tatsächliche Befehl / die tatsächliche Assertion für das angegebene Element.

## Installation

Installieren Sie Node.js
Von nodejs.org
"Node.js ist eine Plattform, die auf der JavaScript-Laufzeitumgebung von Chrome basiert und auf einfache Weise schnelle, skalierbare Netzwerkanwendungen erstellt. Node.js verwendet ein ereignisgesteuertes, nicht blockierendes E / A-Modell, das es leicht und effizient macht und ideal für datenintensive Anwendungen ist -time-Anwendungen, die über verteilte Geräte laufen. "

Installationspakete und Anweisungen für die meisten wichtigen Betriebssysteme finden Sie auf der Website nodejs.org . Denken Sie daran, auch das npm- Tool zu installieren , das den Knotenpaket-Manager darstellt und mit dem Installationsprogramm Node.js verteilt wird.
## Nightwatch installieren

npmFühren Sie Folgendes aus, um die neueste Version mithilfe des Befehlszeilentools zu installieren :

`npm install nightwatch`

* -gOption hinzufügen, um den nightwatchRunner global in Ihrem System verfügbar zu machen.
* --save-devOption hinzufügen, um sie nightwatchals devDependencyin Ihrem package.json zu speichern.
## WebDriver Service

WebDriver Binary Browser
Beschreibung
GeckoDriver	Mozilla Firefox	Standalone-Server, der das W3C WebDriver-Protokoll für die Kommunikation mit Gecko-Browsern wie Firefox implementiert.

Chrome-Treiber	Google Chrome	Standalone-Server, der das JSON Wire-Protokoll für Chromium implementiert , derzeit wird jedoch auf die W3C-WebDriver-Spezifikation umgestellt.

Verfügbar für Chrome für Android und Chrome für Desktop (Mac, Linux, Windows und Chrome OS).
Microsoft WebDriver	Microsoft Edge	Windows-Programmdatei, die sowohl die W3C-WebDriver-Spezifikation als auch das JSON-Drahtprotokoll zum Ausführen von Tests gegen Microsoft Edge unterstützt.
SafariDriver	Microsoft Edge	Die /usr/bin/safaridriverBinärdatei ist mit den neuesten Versionen von Mac OS vorinstalliert und kann gemäß den Anweisungen auf der Apple Developer-Website verwendet werden.

Weitere Informationen finden Sie auf der Seite About WebDriver for Safari .

## WebDriver installieren

Die Installation der WebDriver-Dienste kann entweder durch direktes Herunterladen der Binärdatei oder mithilfe eines NPM-Pakets erfolgen.
### GeckoDriver

GeckoDriver kann von der Release-Seite von GitHub heruntergeladen werden . Dort sind auch Versionshinweise verfügbar. Oder Sie können das Geckodriver- NPM-Paket als Abhängigkeit in Ihrem Projekt verwenden:

`npm install geckodriver --save-dev`
### Chrome-Treiber

ChromeDriver kann von der ChromeDriver-Downloadseite heruntergeladen werden. Oder Sie können das chromedriver NPM-Paket als Abhängigkeit in Ihrem Projekt verwenden:

`npm install chromedriver --save-dev`
### Microsoft WebDriver

WebDriver für Microsoft Edge ist jetzt ein Windows-Feature-on-Demand. Zur Installation führen Sie Folgendes in einer Eingabeaufforderung mit erhöhten Rechten aus:

DISM.exe / Online / Add-Capability /CapabilityName:Microsoft.WebDriver~~~~.0.0.1.0
Weitere Informationen zur Installations- und Verwendungsdokumentation finden Sie auf der offiziellen Microsoft WebDriver-Homepage.
## Selenium Server verwenden

Die Verwendung von Selenium Standalone Server war früher der De-Faktor-Standard für die Verwaltung der verschiedenen Browsertreiber und -dienste. Das Starten mit Nightwatch 1.0 ist jedoch nicht mehr erforderlich und wird auch nicht empfohlen, es sei denn, Sie testen mit älteren Browsern, z. B. Internet Explorer.

Dies kann erforderlich sein, wenn Sie eine Umgebung mit Selen-Gitter haben.
## Laden Sie Java herunter

Selenium Server ist eine Java-Anwendung. Das bedeutet, dass Sie das Java Development Kit (JDK) installiert haben müssen. Die Mindestversion ist 7. Sie können dies überprüfen, indem Sie java -versionvon der Befehlszeile aus starten.
## Laden Sie Selenium herunter

Laden Sie die neueste Version der selenium-server-standalone-{VERSION}.jarDatei von der Seite mit den Selenium-Downloads herunter und platzieren Sie sie mit dem Browser, den Sie testen möchten, auf dem Computer. In den meisten Fällen ist dies auf Ihrem lokalen Computer und normalerweise im Quellordner des Projekts.

Es empfiehlt sich, einen separaten Unterordner (z. B. bin) zu erstellen und dort abzulegen, da möglicherweise andere Treiber-Binärdateien heruntergeladen werden müssen, wenn Sie mehrere Browser testen möchten.

## Selen automatisch laufen lassen

Wenn sich der Server auf demselben Computer befindet, auf dem Nightwatch ausgeführt wird, kann er vom Nightwatch Test Runner direkt gestartet / gestoppt werden .

## Selen manuell laufen lassen

Um den Selenium Server manuell auszuführen, führen Sie im Verzeichnis mit dem jar Folgendes aus:

`java -jar selenium-server-standalone-{VERSION}.jar`

## Selenium Standalone Server verwenden

Führen Sie zum Anzeigen aller Laufzeitoptionen den vorherigen Befehl aus und fügen Sie Folgendes hinzu -help:

`java -jar selenium-server-standalone-{VERSION}.jar -help`

INFO:
Firefox-Benutzer sollten GeckoDriver für ihre Tests verwenden. Weitere Informationen finden Sie im Abschnitt zum Einrichten des Browsers.

## Aufbau

Die `nightwatch` Testrunner-Binärdatei erwartet eine Konfigurationsdatei, die standardmäßig eine `nightwatch.js` onDatei aus dem aktuellen Arbeitsverzeichnis verwendet. Eine `nightwatch.conf.js` Datei wird standardmäßig auch geladen, wenn sie gefunden wird.

## Nachtwache.json

Zu diesem Zeitpunkt sollten Sie mindestens einen WebDriver-Dienst in Ihrem Projekt eingerichtet haben.

Erstellen Sie das `nightwatch.json` im Stammordner des Projekts.

Unter der Annahme, dass Sie den ChromeDriver-Dienst heruntergeladen oder installiert haben, sieht die einfachste `nightwatch.js` onDatei wie `node_modules/.bin/chromedriverfolgt` aus: Wo ist der Pfad, in dem ChromeDriver installiert ist:

```s
{
  "src_folders" : ["tests"],

  "webdriver" : {
    "start_process": true,
    "server_path": "node_modules/.bin/chromedriver",
    "port": 9515
  },

  "test_settings" : {
    "default" : {
      "desiredCapabilities": {
        "browserName": "chrome"
      }
    }
  }
}
```

Die Verwendung beider Konfigurationsdateien ist auch möglich, wobei `nightwatch.conf.js` immer Vorrang hat, wenn beide gefunden werden.

Quelle:

* [nightwatchjs](http://nightwatchjs.org/gettingstarted/)

## [puppeteer](../puppeteer)

Most things that you can do manually in the browser can be done using Puppeteer!
Here are a few examples to get you started:

    Generate screenshots and PDFs of pages.
    Crawl a SPA (Single-Page Application) and generate pre-rendered content (i.e. "SSR" (Server-Side Rendering)).
    Automate form submission, UI testing, keyboard input, etc.
    Create an up-to-date, automated testing environment. Run your tests directly in the latest version of Chrome using the latest JavaScript and browser features.
    Capture a timeline trace of your site to help diagnose performance issues.
    Test Chrome Extensions.

**Quelle:**

* [puppeteer](https://pptr.dev/)

## [testcafe](../testcafe)

Testen Sie alles und überall
Vom Download bis zur Aufnahme Ihres ersten Tests in weniger als 5 Minuten - das Installationsprogramm konfiguriert Ihre Umgebung automatisch.
Mit TestCafe können Sie Tests in jedem Browser ausführen, der HTML5 unterstützt (einschließlich IE9 +, Chrome, Firefox, Safari, Opera).
TestCafe ist unabhängig vom Betriebssystem, so dass Sie Tests auf Windows-, Mac- oder Linux-Computern ausführen können.
Führen Sie Tests auf Remote-Computern und mobilen Geräten durch.
Führen Sie Tests in mehreren Browsern und auf mehreren Computern parallel aus.
Führen Sie Tests im Hintergrund auf einem beliebigen Computer aus.
Mit TestCafe können Sie Webseiten testen, für die die Basis- und die Windows-HTTP-Authentifizierung erforderlich sind.

### Quelle

* [testcafe](https://testcafe.devexpress.com/)

### Tools

* [Test Automatiesirung](test-automation.md)