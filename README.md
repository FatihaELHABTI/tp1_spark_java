
# TP Spark Java — Analyse de données (Ventes & Logs Apache)

## 🎯 Objectif du Projet
Ce projet permet de pratiquer **Apache Spark en Java** via deux exercices :

- **Exercice 1 : Analyse des ventes**  
  Traitement d’un fichier `ventes.txt` pour calculer les ventes par ville et par année.

- **Exercice 2 : Analyse de logs Apache**  
  Extraction d’informations depuis `access.log` (IP, ressource, code HTTP, erreurs, statistiques…).

Le tout fonctionne **en local** ou via un **cluster Hadoop + Spark Dockerisé**.

---

# 🛠️ 1. Structure du Projet

```
tp1-spark-java/
│── src/main/java/ma/enset/
│   ├── Exercice1.java
│   └── Exercice2.java
│── data/
│   ├── ventes.txt
│   └── access.log
│── pom.xml
│── docker-compose.yaml
│── config/ (fichiers core-site, yarn-site…)
```

---

# 🚀 2. Exécution avec Spark local ou Hadoop + Docker

### ✅ Mode local
Spark lit les fichiers depuis `data/`.

```java
JavaRDD<String> lines = sc.textFile("data/ventes.txt");
```

### ✅ Mode cluster Hadoop/Spark
Spark lit depuis **HDFS** :

```java
JavaRDD<String> lines = sc.textFile("hdfs://namenode:8020/data/ventes.txt");
```

---

# 📦 3. Configuration Maven (pom.xml)

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.13</artifactId>
    <version>${spark.version}</version>
</dependency>
```

- JDK : **17**
- Spark Core : **4.0.1**
- Mode Standalone ou cluster Hadoop/Spark

---

# 📊 4. Exercice 1 — Analyse des ventes

## 📝 4.1. Chargement des données

Chaque ligne du fichier `ventes.txt` :

```
2024-05-10 Paris Ordinateur 1200.50
```

Découpage en Spark :

```java
String[] parts = line.split(" ");
String ville = parts[1];
double prix = Double.parseDouble(parts[3]);
```

---

## 📍 4.2. Total des ventes par ville

```java
JavaPairRDD<String, Double> ventesParVille =
    lines.mapToPair(line -> new Tuple2<>(ville, prix));

JavaPairRDD<String, Double> totalParVille =
    ventesParVille.reduceByKey(Double::sum);
```

Sortie typique :

```
Ville: Paris, Total: 2725.99
Ville: Lyon, Total: 2775.00
Ville: Marseille, Total: 1580.00
```

---

## 📅 4.3. Total par ville et par année

```java
String annee = parts[0].substring(0, 4);
return new Tuple2<>(new Tuple2<>(annee, ville), prix);
```

---

# 📑 5. Exercice 2 — Analyse de Logs Apache

## 🧩 5.1. Format du fichier `access.log`

Une ligne exemple :

```
127.0.0.1 - - [10/Oct/2025:09:15:32 +0000] "GET /index.html HTTP/1.1" 200 1024
```

### 📌 Extraction via Regex

```java
private static final String LOG_REGEX =
  "^(\S+) (\S+) (\S+) \[([\w:/]+\s[+\-]\d{4})\] \"(\S+) (\S+) (\S+)\" (\d{3}) (\d+)";
```

Classe représentant une entrée :

```java
public static class LogEntry implements Serializable {
    String ip, dateTime, method, resource;
    int httpCode;
    long responseSize;
}
```

---

## 📊 5.2. Statistiques globales

### ✔ Nombre total de requêtes  
```java
long totalRequests = parsedLogs.count();
```

### ✔ Nombre d’erreurs HTTP (>=400)

```java
parsedLogs.filter(log -> log.httpCode >= 400).count();
```

---

## 🥇 5.3. Top 5 adresses IP

```java
parsedLogs.mapToPair(log -> new Tuple2<>(log.ip, 1))
          .reduceByKey(Integer::sum)
```

---

## 🥇 5.4. Top 5 ressources les plus consultées

```java
parsedLogs.mapToPair(log -> new Tuple2<>(log.resource, 1))
```

---

## 📈 5.5. Répartition des codes HTTP

```java
parsedLogs.mapToPair(log -> new Tuple2<>(log.httpCode, 1))
```

---

# 🐳 6. Cluster Hadoop + Spark (docker-compose.yaml)

Le projet inclut :

- **Hadoop NameNode / DataNode**
- **YARN ResourceManager / NodeManager**
- **Spark Master / Worker**
- Réseau : `spark-network`

L'application Spark se connecte automatiquement à :

```
hdfs://namenode:8020
```

---

# ⚙️ 7. Fichiers de configuration Hadoop (config/)

Exemples :

### core-site.xml
```
fs.defaultFS=hdfs://namenode
```

### hdfs-site.xml
```
dfs.replication=3
```

### yarn-site.xml
```
yarn.resourcemanager.hostname=resourcemanager
```

---

# 📁 8. Jeux de données

## ✔ ventes.txt
Contient : date, ville, produit, prix.

## ✔ access.log
Contient : IP, date, méthode HTTP, ressource, code HTTP, taille réponse.

---

# 🏁 9. Conclusion

Ce TP met en pratique :

- Spark Core (RDD)
- Transformations & actions (map, reduceByKey, filter…)
- Analyse de fichiers structurés (ventes) et semi‑structurés (logs)
- Exécution Spark standalone ou cluster Hadoop/Spark
- Intégration HDFS pour les datasets

Ce README permet de comprendre clairement la logique du code, les concepts Spark utilisés et la configuration complète du cluster.

---

# 👤 Auteur
TP réalisé par **Fatiha EL HABTI** dans le cadre du module **Big Data - Spark**.
