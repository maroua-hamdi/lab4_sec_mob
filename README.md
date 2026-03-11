# 🔐 Analyse statique d'une application Android

## Lab — OWASP UnCrackable Level 1 

## 📚 Objectif du lab

L’objectif de ce lab est de réaliser une **analyse statique d’une application Android (APK)** afin de comprendre son fonctionnement interne et d’identifier d’éventuelles faiblesses de sécurité.

L’application étudiée est :

```
OWASP MSTG UnCrackable Level 1
```

Le but est de **retrouver le secret caché dans l’application** en analysant son code sans modifier l’APK.

---

# 🛠 Outils utilisés

Les outils suivants ont été utilisés durant ce lab :

* **JADX GUI** — Décompilation d’APK
* **PowerShell** — Analyse et extraction
* **Android Emulator** — Test de l’application
* **dex2jar** — Conversion DEX → JAR
* **JD-GUI** — Lecture du code Java

---

# 📂 1 — Préparation de l’environnement

Un dossier de travail a été créé afin d'organiser l'analyse de l'APK.

Commande utilisée :

```powershell
mkdir C:\APK-Analysis
cd C:\APK-Analysis
```

L’APK a ensuite été copié dans ce dossier :

```
UnCrackable-Level1.apk
```

<img width="899" height="486" alt="pic 1" src="https://github.com/user-attachments/assets/3d6247fc-027d-4344-b1b1-bb980f315129" />

```


---

# 📦 2 — Vérification du contenu de l’APK

Une application Android est en réalité **une archive ZIP** contenant :

* le code compilé
* les ressources
* le manifest Android

Pour vérifier son contenu, la commande suivante a été utilisée :

```powershell
Add-Type -Assembly System.IO.Compression.FileSystem
$apk = Join-Path (Get-Location).Path "UnCrackable-Level1.apk"
[System.IO.Compression.ZipFile]::OpenRead($apk).Entries | Select-Object -ExpandProperty FullName -First 20
```

Résultat observé :

```
AndroidManifest.xml
META-INF/
classes.dex
res/
resources.arsc
```

<img width="1187" height="365" alt="pic 2" src="https://github.com/user-attachments/assets/c6021b01-6791-466d-81f2-e0143b30db86" />


# 🔑 3 — Vérification de l’intégrité (SHA256)

Pour garantir l’intégrité de l’APK, un **hash SHA256** a été calculé.

Commande utilisée :

```powershell
Get-FileHash -Algorithm SHA256 .\UnCrackable-Level1.apk
```

Résultat obtenu :

```
1DA8BF57D266109F9A07C01BF7111A1975CE01F190B9D914BCD3AE3DBEF96F21
```

<img width="1262" height="172" alt="pic 3" src="https://github.com/user-attachments/assets/c5087d03-9045-4598-8fd0-de21b665e742" />


# 🔍 4 — Analyse statique avec JADX

L’APK a été ouvert avec **JADX GUI** afin de décompiler l’application.

Étapes :

```
File → Open → UnCrackable-Level1.apk
```

JADX permet d’accéder au code Java généré à partir du bytecode Android.

Package principal identifié :

```
sg.vantagepoint.uncrackable1
```

Classes importantes :

```
MainActivity
a
```
<img width="1705" height="685" alt="pic 4" src="https://github.com/user-attachments/assets/8268c839-aeab-4b78-8bbc-41ce4ca2632e" />



Ce screenshot doit montrer :

* la structure de l’APK
* le code Java décompilé

---

# 🧠 5 — Analyse du code

Dans la classe :

```
sg.vantagepoint.uncrackable1.a
```

une fonction de vérification a été identifiée :

```java
public static boolean a(String str)
```

Cette fonction compare l’entrée utilisateur avec une valeur décryptée :

```java
return str.equals(new String(bArr));
```

Les données suivantes ont été trouvées dans le code :

```
8d127684cbc37c17616d806cf50473cc
5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR260c=
```

Ces valeurs sont utilisées pour :

* décodage **Base64**
* déchiffrement **AES**

📸 **Screenshot à ajouter ici :**

```
screenshots/code_secret.png
```

Ce screenshot doit montrer la fonction contenant la vérification du secret.

---

# 📦 6 — Extraction du fichier DEX

L’APK a été extrait afin d’obtenir le fichier contenant le bytecode Android.

Commande utilisée :

```powershell
tar -xf UnCrackable-Level1.apk
```

Fichiers obtenus :

```
classes.dex
AndroidManifest.xml
resources.arsc
res/
META-INF/
```
<img width="1008" height="707" alt="pic 5" src="https://github.com/user-attachments/assets/fadbc83a-4b4b-47fd-ac03-a1d8556aad76" />


# 🔄 7 — Conversion DEX → JAR

Le fichier **classes.dex** peut être converti en **JAR** pour être analysé avec des outils Java.

Commande utilisée :

```powershell
d2j-dex2jar.bat classes.dex -o app.jar
```

Cela permet d’ouvrir le code avec **JD-GUI**.

# 🔐 Analyse des méthodes de vérification et de chiffrement

Avant d’obtenir le résultat final, il est important d’analyser les méthodes responsables de la **vérification du secret et du traitement de la clé chiffrée**.

## Méthode de vérification du secret

Dans la classe `MainActivity`, la méthode suivante est appelée lorsque l’utilisateur clique sur le bouton de vérification :

```java
public void verify(View view)
```

Cette méthode récupère le texte saisi par l’utilisateur et appelle la fonction de vérification :

```java
if (a.a(string)) {
    alertDialogCreate.setTitle("Success!");
    str = "This is the correct secret.";
}
```

La fonction `a.a(string)` compare l’entrée utilisateur avec la valeur correcte générée par le programme.

<img width="1200" height="367" alt="pic 6" src="https://github.com/user-attachments/assets/b3949d12-9d3a-4c00-9f8a-31478f7ecde0" />


---

## Méthode de conversion hexadécimale

Une autre méthode importante identifiée dans le code est :

```java
public static byte[] b(String str)
```

Cette fonction convertit une chaîne **hexadécimale** en tableau de bytes.
Elle est utilisée pour préparer la clé utilisée dans le processus de déchiffrement.

Extrait du code :

```java
public static byte[] b(String str) {
    int length = str.length();
    byte[] bArr = new byte[length / 2];
    for (int i = 0; i < length; i += 2) {
        bArr[i / 2] = (byte) ((Character.digit(str.charAt(i), 16) << 4)
        + Character.digit(str.charAt(i + 1), 16));
    }
    return bArr;
}
```

Cette fonction transforme donc une **clé hexadécimale en format exploitable par l’algorithme de chiffrement**.

<img width="1016" height="170" alt="pic 7" src="https://github.com/user-attachments/assets/9349d31a-6b24-458b-b7bf-826242999982" />




# 📱 8 — Exécution de l’application

L’application a été exécutée dans un **Android Emulator**.

Après analyse du code, le secret correct a été trouvé.

Secret entré :

```
I want to believe
```

Message affiché par l’application :

```
Success!
This is the correct secret.
```

<img width="947" height="771" alt="pic 9" src="https://github.com/user-attachments/assets/11ac0b69-3923-4f6f-ae93-04ad68fdb92a" />


Ce screenshot doit montrer :

* le secret entré
* le message **Success**

---

# 🎯 Résultat final

Le secret caché dans l’application est :

```
I want to believe
```

---

# ⚠ Problème de sécurité identifié

L’application contient un :

```
Hardcoded Secret
```

Cela signifie que le secret est directement stocké dans le code client.

Un attaquant peut facilement le récupérer en :

1. décompilant l’APK
2. analysant le code

---

# 🛡 Recommandations

Pour corriger ce problème :

* ne pas stocker de secrets dans le code client
* effectuer la vérification côté serveur
* utiliser un système d’authentification sécurisé

---

# 📊 Conclusion

Ce lab démontre qu’une application Android peut être analysée et comprise à l’aide d’outils de reverse engineering.

L’analyse statique permet notamment de :

* comprendre la structure d’un APK
* décompiler du bytecode Android
* identifier des failles de sécurité
* récupérer des secrets cachés


---

# 🔗 Référence

OWASP Mobile Security Testing Guide

https://mas.owasp.org/
