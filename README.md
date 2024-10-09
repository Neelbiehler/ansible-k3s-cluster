# Ansible K3s Cluster auf Hetzner-Servern

Dieses Repository enthält ein erweitertes Ansible-Playbook zur Einrichtung eines produktionsreifen K3s-Clusters auf Hetzner-Servern. Das Playbook berücksichtigt Best Practices für Sicherheit, Skalierbarkeit und Hochverfügbarkeit und integriert zahlreiche Komponenten, um eine umfassende Produktionsumgebung bereitzustellen.

## Inhaltsverzeichnis

- [Voraussetzungen](#voraussetzungen)
- [Projektstruktur](#projektstruktur)
- [Enthaltene Rollen](#enthaltene-rollen)
- [Installation](#installation)
- [Verwendung](#verwendung)
- [Komponenten im Detail](#komponenten-im-detail)
  - [Hetzner Netzwerk und Load Balancer](#hetzner-netzwerk-und-load-balancer)
  - [K3s Cluster](#k3s-cluster)
  - [Netzwerkmanagement mit Cilium](#netzwerkmanagement-mit-cilium)
  - [Monitoring und Logging](#monitoring-und-logging)
  - [Zertifikatsverwaltung](#zertifikatsverwaltung)
  - [Identity Management mit Keycloak](#identity-management-mit-keycloak)
  - [Datenbanken und Caching](#datenbanken-und-caching)
  - [Anwendungen](#anwendungen)
  - [CI/CD und Automatisierung](#cicd-und-automatisierung)
- [Best Practices](#best-practices)
- [Ressourcen](#ressourcen)
- [Lizenz](#lizenz)

## Voraussetzungen

- **Ansible** installiert auf Ihrer lokalen Maschine
- **Hetzner Cloud API Token** mit ausreichenden Berechtigungen
- **Domain Name** für Ingress und Zertifikate
- **E-Mail-Adresse** für Let's Encrypt Zertifikate
- **Container Registry Zugangsdaten** für das Pushen von Docker-Images
- **SSH-Zugang** zu Ihren Hetzner-Servern
- **Ansible Vault** (empfohlen für das sichere Speichern von sensiblen Daten)

## Projektstruktur

```bash
ansible-k3s-cluster/
├── playbook.yml
├── vars.yml
├── inventory/
│   └── hosts.ini
└── roles/
    ├── hetzner-network/
    ├── hetzner-loadbalancer/
    ├── k3s/
    ├── cilium/
    ├── network-policies/
    ├── nginx-ingress/
    ├── prometheus-grafana/
    ├── logging/
    ├── fluentd/
    ├── cert-manager/
    ├── keycloak-idp/
    ├── postgresql/
    ├── redis-cluster/
    ├── model-database/
    ├── nextjs-frontend/
    ├── hono-backend/
    ├── python-model/
    ├── hpa/
    ├── velero/
    ├── vault/
    ├── istio/
    ├── healthchecks/
    ├── disaster-recovery/
    ├── documentation/
    ├── security-scanning/
    ├── update-strategy/
    └── ci-cd/
```

### Enthaltene Rollen

- **hetzner-network**: Einrichtung eines privaten Netzwerks in der Hetzner Cloud.
- **hetzner-loadbalancer**: Konfiguration eines Load Balancers für Traffic-Verteilung.
- **k3s**: Installation und Konfiguration von K3s.
- **cilium**: Installation des Cilium CNI für Netzwerkmanagement.
- **network-policies**: Anwendung von Netzwerk-Richtlinien für erhöhte Sicherheit.
- **nginx-ingress**: Installation des NGINX Ingress Controllers.
- **prometheus-grafana**: Einrichtung von Monitoring mit Prometheus und Grafana.
- **logging**: Installation eines Logging-Stacks (Loki).
- **fluentd**: Erweiterung des Logging-Systems mit Fluentd.
- **cert-manager**: Automatisierte Zertifikatsverwaltung mit cert-manager.
- **keycloak-idp**: Installation von Keycloak als Identity Provider.
- **postgresql**: Einrichtung einer PostgreSQL-Datenbank mit Replikation.
- **redis-cluster**: Installation eines Redis-Clusters für Caching.
- **model-database**: Einrichtung einer MongoDB-Datenbank mit Replikation.
- **nextjs-frontend**: Deployment einer Next.js Frontend-Anwendung.
- **hono-backend**: Deployment des Hono API Backends.
- **python-model**: Deployment eines Python-Modells für Datenverarbeitung.
- **hpa**: Konfiguration von Horizontal Pod Autoscaling.
- **velero**: Implementierung einer Backup-Strategie mit Velero.
- **vault**: Secrets Management mit HashiCorp Vault.
- **istio**: Integration des Istio Service Mesh.
- **healthchecks**: Einrichtung von Healthchecks und Readiness Probes.
- **disaster-recovery**: Planung und Dokumentation von Disaster Recovery.
- **documentation**: Automatische Generierung von Cluster-Dokumentation.
- **security-scanning**: Integration von Sicherheits-Scanning mit Trivy.
- **update-strategy**: Definition einer Update- und Rollback-Strategie.
- **ci-cd**: Integration einer CI/CD-Pipeline.


## Installation

1. Repository klonen:
   ```bash
   git clone https://github.com/neelbiehler/ansible-k3s-cluster.git
   cd ansible-k3s-cluster
   ```

2. Voraussetzungen installieren:
   Stellen Sie sicher, dass Ansible und die benötigten Ansible Galaxy Rollen installiert sind:
   ```bash
   ansible-galaxy collection install community.general
   ```

3. Variablen konfigurieren:
   Erstellen Sie eine `vars.yml` Datei und fügen Sie Ihre spezifischen Variablen hinzu:
   ```yaml
   hetzner_token: "Ihr_Hetzner_API_Token"
   domain_name: "example.com"
   email_address: "admin@example.com"
   registry_username: "Ihr_Registry_Benutzername"
   registry_password: "Ihr_Registry_Passwort"
   ```
   > **Hinweis:** Speichern Sie diese Datei sicher und verwenden Sie Ansible Vault für sensible Daten.

4. Inventardatei anpassen:
   Bearbeiten Sie die `inventory/hosts.ini` Datei und fügen Sie Ihre Server hinzu:
   ```ini
   [all]
   node1 ansible_host=Master_Server_IP
   node2 ansible_host=Worker_Server_IP

   [masters]
   node1

   [workers]
   node2
   ```

5. Playbook ausführen:
   ```bash
   ansible-playbook -i inventory/hosts.ini playbook.yml
   ```

## Verwendung

- **Überwachung:** Nach der Installation können Sie auf Grafana zugreifen unter `https://grafana.example.com`.
- **Anwendungen:** Ihre Anwendungen sind über die konfigurierten Ingress-Routen erreichbar.
- **Verwaltung:** Verwenden Sie `kubectl`, um Ihren Cluster zu verwalten. Die kubeconfig-Datei befindet sich in Ihrem Home-Verzeichnis unter `~/.kube/config`.

## Komponenten im Detail

### Hetzner Netzwerk und Load Balancer
- **Private Netzwerke:** Ermöglichen sichere interne Kommunikation zwischen den Knoten.
- **Load Balancer:** Verteilt eingehenden Traffic und erhöht die Verfügbarkeit.

### K3s Cluster
- **Leichtgewichtig:** K3s ist eine minimalistische Kubernetes-Distribution.
- **Hochverfügbar:** Durch die Konfiguration von mehreren Master- und Worker-Knoten.

### Netzwerkmanagement mit Cilium
- **eBPF:** Cilium nutzt eBPF für effizientes Netzwerkmanagement.
- **Sicherheit:** Unterstützt Netzwerk-Richtlinien für verbesserte Sicherheit.

### Monitoring und Logging
- **Prometheus und Grafana:** Überwachung der Cluster-Leistung und Ressourcen.
- **Loki und Fluentd:** Sammlung und Analyse von Logs.

### Zertifikatsverwaltung
- **cert-manager:** Automatisiert die Erstellung und Erneuerung von TLS-Zertifikaten.
- **Let's Encrypt:** Kostenlose SSL-Zertifikate für Ihre Domains.

### Identity Management mit Keycloak
- **SSO:** Bietet Single Sign-On und Identity Federation.
- **Integration:** Kann mit Ihren Anwendungen für Authentifizierung verwendet werden.

### Datenbanken und Caching
- **PostgreSQL und MongoDB:** Hochverfügbare Datenbanklösungen mit Replikation.
- **Redis-Cluster:** Schnelles Caching und Datenpersistenz.

### Anwendungen
- **Next.js Frontend:** Modernes React-Framework für Ihre Benutzeroberfläche.
- **Hono API Backend:** Leistungsfähiges Backend für Ihre Anwendungen.
- **Python-Modell:** Verarbeitung von Daten und Machine-Learning-Modelle.

### CI/CD und Automatisierung
- **CI/CD-Pipeline:** Automatische Builds und Deployments Ihrer Anwendungen.
- **Vault:** Sicheres Management von Geheimnissen und Zugangsdaten.
- **Istio Service Mesh:** Erweiterte Verkehrssteuerung und Sicherheit.

## Best Practices

- **Sicherheit:** Verwenden Sie Ansible Vault für sensible Daten und implementieren Sie Netzwerk-Richtlinien.
- **Skalierbarkeit:** Nutzen Sie HPA und Ressourcenlimits, um effizient zu skalieren.
- **Überwachung:** Behalten Sie Ihr System mit Prometheus und Grafana im Blick.
- **Backups:** Implementieren Sie regelmäßige Backups mit Velero.
- **Dokumentation:** Halten Sie Ihre Dokumentation aktuell für eine einfachere Wartung.

## Ressourcen

- [Ansible Dokumentation](https://docs.ansible.com/)
- [Kubernetes Dokumentation](https://kubernetes.io/docs/)
- [Hetzner Cloud API](https://docs.hetzner.cloud/)
- [Helm Charts Repository](https://artifacthub.io/)

## Lizenz

Dieses Projekt ist frei von Copyright und Lizenzbeschränkungen. Es kann von jedem ohne Einschränkungen oder Verpflichtungen genutzt, kopiert, modifiziert, verteilt oder für jeden beliebigen Zweck verwendet werden.

Es gibt keine Garantien oder Haftungen jeglicher Art. Die Nutzung erfolgt auf eigenes Risiko.

Jeder ist eingeladen, dieses Projekt frei zu verwenden und zu teilen.

