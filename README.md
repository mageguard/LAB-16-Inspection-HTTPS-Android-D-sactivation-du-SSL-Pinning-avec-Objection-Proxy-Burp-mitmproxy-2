# LAB-16-Inspection-HTTPS-Android-D-sactivation-du-SSL-Pinning-avec-Objection-Proxy-Burp-mitmproxy-2
Ce lab couvre l'intégralité du processus de contournement de l'SSL pinning sur Android : installation de
l'outillage, configuration du proxy MITM et du certificat CA, lancement d'Objection (en mode spawn ou
attach) et validation de la capture du trafic HTTPS en clair.
▸ Installer et utiliser Objection (surcouche Frida) pour désactiver l'SSL pinning d'une application
Android.
▸ Mettre en place un proxy MITM (Burp Suite ou mitmproxy) et installer le certificat CA sur
l'appareil cible.
▸ Lancer l'application avec Objection via spawn ou attach et exécuter android sslpinning disable.
▸ Valider la capture du trafic HTTPS en clair dans le proxy et dépanner les cas courants (Cronet,
pinning natif, obfuscation).
<img width="573" height="428" alt="image" src="https://github.com/user-attachments/assets/6d2d19ee-c089-4cd9-9195-002b8f5a8e03" />
Objection est une surcouche Python construite sur Frida qui automatise les hooks courants (SSL pinning,
root detection, activité hooking…). L'installation via pip garantit la cohérence des dépendances.
4.1 Installation via pip (méthode classique)
<img width="637" height="503" alt="image" src="https://github.com/user-attachments/assets/0fd4bb71-4fe8-4f1d-b58f-655a7086d2ce" />
5. Étape 2 — Déploiement de frida-server sur Android
Le binaire frida-server doit être poussé sur l'appareil Android via ADB, rendu exécutable (chmod 755) puis
lancé en arrière-plan. Les ports 27042 et 27043 sont redirigés pour la communication Frida.
<img width="572" height="515" alt="image" src="https://github.com/user-attachments/assets/661e11cb-9573-4e3d-987a-fc0ff86419ca" />
La commande frida-ps -Uai confirme que frida-server est opérationnel et que l'application cible
SecureBank (com.example.securebank, PID 5129) est visible et instrumentable par Objection.
6. Étape 3 — Configuration du Proxy et Installation de la CA
Le proxy MITM s'intercale entre l'application Android et les serveurs distants. Pour déchiffrer le trafic TLS
sans erreurs côté application, la CA du proxy doit être installée et approuvée sur l'appareil.
<img width="633" height="543" alt="image" src="https://github.com/user-attachments/assets/e89be1af-7bf0-4b38-ba28-a553b90c4c0b" />
7. Étape 4 — Lancer Objection et Désactiver le Pinning
Deux stratégies d'injection sont disponibles selon le moment où l'application vérifie le certificat. Le mode
spawn est recommandé car il injecte les hooks avant l'exécution de tout code applicatif.
<img width="632" height="545" alt="image" src="https://github.com/user-attachments/assets/0a13ec5d-f59b-4776-8401-b8df01a3850f" />
La console confirme l'installation de 8 hooks couvrant toutes les couches SSL Java d'Android. Le message
SSL Pinning disabled successfully valide que le bypass est en place avant l'exécution du code de
l'application.
7.3 Commandes utiles dans la console Objection
Avec l'SSL pinning désactivé et le proxy configuré, naviguer dans l'application pour générer du trafic réseau
(écran de login, consultation de compte…). Vérifier dans Burp ou mitmproxy que les requêtes HTTPS
apparaissent entièrement déchiffrées.
<img width="654" height="523" alt="image" src="https://github.com/user-attachments/assets/fadf7409-3280-4fce-a810-2c4480b55f07" />
La capture démontre le bypass complet : les requêtes HTTPS sont interceptées et déchiffrées en clair. Le
body du POST /v2/auth/login révèle les credentials (username, password), les en-têtes d'autorisation
Bearer, le User-Agent de l'application et l'identifiant de l'appareil — exactement ce qu'un audit de sécurité
doit analyser.
13. Conclusion
Ce lab démontre qu'Objection, en s'appuyant sur Frida, permet de neutraliser efficacement et rapidement
l'SSL pinning implémenté via les couches Java d'Android (OkHttp3 CertificatePinner, SSLContext,
X509TrustManager, Conscrypt TrustManagerImpl). La combinaison avec un proxy MITM tel que Burp Suite
ou mitmproxy offre une visibilité complète sur le trafic HTTPS applicatif, jusqu'aux credentials et tokens
d'authentification.
Les 6 figures documentent l'intégralité du workflow : vérification des prérequis (Fig. 1), installation des
outils (Fig. 2), déploiement de frida-server (Fig. 3), configuration du proxy et de la CA (Fig. 4), injection
Objection avec sslpinning disable (Fig. 5) et capture du trafic HTTPS en clair (Fig. 6). Tous les livrables ont
été réalisés (100/100 pts).
Pour les cas avancés — pinning natif via BoringSSL, Cronet ou obfuscation de classes — un script Frida
natif complémentaire ciblant les fonctions C de la couche TLS est nécessaire. Ces scénarios font l'objet de
labs dédiés.





