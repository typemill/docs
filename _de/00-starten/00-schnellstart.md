# Quickstart

Starten Sie schnell und einfach mit Typemill durch! Diese Anleitung hilft Ihnen, Ihr erstes Projekt im Handumdrehen einzurichten.

## Typemill installieren

Wenn Sie bereits eine PHP-basierte Website haben, sollte die Installation von Typemill weniger als fünf Minuten dauern. Sie können Typemill manuell, über GitHub oder mit DockerHub installieren. Folgen Sie einfach der [kurzen Installationsanleitung](/getting-started/installation).

## Erstellen Sie Ihren ersten Benutzer

Nach der Installation besuchen Sie die URL Ihrer Website. Sie werden zu einem Einrichtungsformular weitergeleitet, um Ihr Benutzerkonto mit Benutzernamen, E-Mail und Passwort zu erstellen.

## Anmelden

Sobald Ihr Konto erstellt ist, können Sie sich unter der URL `/tm/login` im System anmelden.

## Konfigurieren Sie das System

Bei Ihrer ersten Anmeldung werden Sie zu den Systemeinstellungen weitergeleitet. Für einen schnellen Start können Sie die Standardkonfigurationen beibehalten oder Einstellungen für folgende Bereiche anpassen:

* System
* Plugins
* Designs

## Erstellen Sie Inhalte

Unmittelbar nach der Installation haben Sie Zugriff auf eine Demo-Website, die verschiedene Funktionen wie versteckte Seiten, geschützte Seiten und Weiterleitungen zeigt. Melden Sie sich im Autorenbereich (/tm/login) an und navigieren Sie in der oberen Navigation zu „content", um alle Inhalte zu bearbeiten:

* Nutzen Sie die interaktive Drag-and-Drop-Funktion.
* Organisieren Sie Ihre Website mit Ordnern und Seiten.
* Schreiben Sie Inhalte mit Markdown-Syntax.
* Wechseln Sie zwischen dem visuellen Editor und dem Raw-Markdown-Editor.
* Bearbeiten Sie Seitenmeta-Informationen in den Meta-Tabs.
* Erstellen Sie Abschnitte für Nachrichten oder Blog-Beiträge.

## Entwickeln Sie ein Design

Typemill bietet eine Vielzahl von Designs, aber Sie können auch Ihr eigenes mit Twig und YAML erstellen. Beachten Sie die Theme-Entwickleranleitung und verwenden Sie das dev-theme als Ausgangspunkt.

Sie können Variablen und Funktionen für Ihr Design erkunden und Beispiele für Navigation, Artikellisten, Paginierung und Bildbearbeitung ansehen.

## Entwickeln Sie ein Plugin

Das Typemill-Plugin-System nutzt den weit verbreiteten Symfony Event Dispatcher zusammen mit Vue-Komponenten für die Admin-Oberfläche. Erstellen Sie Konfigurationsformulare mit YAML, fügen Sie neue Tabs zum Inhaltsbereich hinzu und führen Sie neue Elemente in die Systemnavigation ein. Aber das ist noch nicht alles:

* Möchten Sie eine neue Benutzerrolle erstellen? Das ist einfach.
* Müssen Sie den Editor erweitern? Schauen Sie sich das Math-Plugin als Beispiel an.
* Interessieren Sie sich für die Integration eines Newsletters? Shortcodes helfen dabei.
* Neue Routen und Middleware hinzufügen? Das ist ebenfalls möglich.

