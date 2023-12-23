![logo_nplus](README.assets/logo_nplus.svg)

Das *nscale Plus Pack* (kurz *nplus*) bietet Werkzeuge und Anleitungen, um das *Enterprise Information Management* System [nscale](https://ceyoniq.com/produkte/) des Herstellers [Ceyoniq](www.ceyoniq.com) in einer **mandantenfähigen** und **hochverfügbaren** Laufzeitumgebung **automatisiert ausrollen** zu können. Zusätzlich werden die original-**Komponenten ergänzt**, um übliche **Konzernanforderungen** abbilden zu können.

*nplus* ist ein **Abonnement**. Während der Laufzeit bekommt der Kunde Zugriff auf alle *nplus* Quellen und kann so einfach seine **Investition schützen**.

>  *nplus* enthält weder Software noch Dienstleistung zu nscale. nscale Lizenzen, nscale Versionen, nscale Support, nscale Consulting etc. sind nach wie vor direkt bei Ceyoniq zu beziehen.



## Eigenschaften

Für den Betrieb in einem Kubernetes Cluster bietet *nplus*:

- Versionierte *helm Charts* für alle *nscale* Komponenten zur Installation, Update und Deinstallation
- Die nscale Komponenten (Application Layer, Storage Layer, Web, CMIS, ...) können in beliebigen Kombinationen zu *Instanzen* zusammengefasst werden.
- Es können mehrere *Instanzen* in einem Kubernetes Namespace laufen (z.B. *Mandant1*, *Mandant2* und *Centralservices*)
- Ein Namespace kann dann ein Mandant sein (z.B. *Sales*) oder eine Stage (z.B. *prod*, *qa* oder *dev*), falls man mehrere Stages auf einem Cluster laufen lassen möchte.
- Umbrella Charts für komplexe Umgebungen, inkl. 
  - optionales LDAP Directory mit openLDAP
  - optionales zentrales Single Sign On inkl. Multi-Faktor-Authentifizierung mit KeyCloak
  - optionale Postgres Datenbank
  - optionale S3 Anbindung für den Storage Layer

- Alle Charts sind auch in einer *argoCD* Variante vorhanden, um diese in eine GitOps Deployment Pipeline zu integrieren.
- Unterstützung von AppDynamics
- Unterstützung von Sicherheits-Werkzeugen, speziell Illumio und Calico
- Unterstützung von snc (zum Zugriff auf SAP Systeme)
- Unterstützung von cert-manager zum automatischen Generieren von tls Zertifikaten
- Ein separates Applikations-Chart (*nplus Application*) zum Einspielen und Aktualisieren von Lösungen
- Verwendbarkeit des klassischen *nscale Administrator*, also keine Umgewöhnungsphase für Ihre nscale Administratoren
- Umbrella Charts mit Mandanten-Vorlagen, die Applikationen, Lösungen und weitere externe Werkzeuge beinhalten und zusammenfassen können. Das Installieren, Deinstallieren oder Updaten solcher Mandanten nach Vorlage ist dann in einer Zeile zu erledigen.
- Verwenden von dedizierten Application Layern für z.B. Jobs, SAP und Anwendern. Für jeden Anwendungsfall können beliebig viele Repliken angegeben werden.
- Verwenden von dedizierten nscale Web Instanzen für z.B. Fachbereich A und Fachbereich B. Für jeden Fachbereich können so z.B. unterschiedliche Snippets geladen werden oder unterschiedliche SSO Regeln gelten.



## Weiterführende Informationen

- Im [Quickstart Guide](docs/quickstart.md) erfahren Sie, wie ein erstes nplus System installiert werden kann.

- [First Day Operations](docs/operations1.md) shows how to install, upgrade and uninstall *nplus Instances*, with and without *argoCD*.  
  Note, that updates can mostly be rolling, except for Major and Minor updates to the *nscale Application Layer*. This document describes both, rolling and recreate updates.

- Have a look at the [samples directory](samples/README.md) to see how Instances get deployed in the *nplus Demo Environment*.

- Pro Komponente gibt es entsprechende Anleitungen im README des Charts.  
  Diese bekommen sie immer aktuell über helm, z.B. für das Instanz-Chart:
  ```
  helm show readme nplus/nplus-instance
  ```
  oder auch hier:
  
- Environment  
  [Environment Chart README](docs/environment.md)  
  
- Instance  
  [Instance Chart README](docs/instance.md)  
  [Instance argoCD Chart README](docs/instance-argo.md)  
  
- Components  
  [nscale Application Layer Chart README](docs/nappl.md)  
  [nscale Storage Layer Chart README](docs/nstl.md)  
  [nscale Pipeliner Chart README](docs/pipeliner.md)  
  [nscale CMIS Connector Chart README](docs/cmis.md)  
  [nscale ILM Connector Chart README](docs/ilm.md)  
  [nscale Web Chart README](docs/web.md)  
  [nscale Monitoring Console Chart README](docs/mon.md)  
  [nscale Rendition Server Chart README](docs/rs.md)  
  
- Application  
  [nplus Application Chart README](docs/application.md)  

