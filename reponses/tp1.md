# Situation 1  :   Une servlet, plusieurs URL

# Le tableu

URL tapée	                        getServletPath()	getPathInfo()	  getQueryString()	Page obtenue
/inscript/modules	                /modules	        null	          null	            liste des 6 modules
/inscript/modules/                /modules	        /	              null	            liste des 6 modules
/inscript/modules/M231	          /modules	        /M231	          null	            détail de M231
/inscript/modules/M231/inscrits	  /modules	        /M231/inscrits	null	            404 « Module inconnu : M231/inscrits »
/inscript/modules?filiere=GM&x=1  /modules	        null	          filiere=GM&x=1	  M311 et M312 (x est ignoré)
/inscript/modules?filiere=XX				_                 _                  _            404 « Filière inconnue : XX »

# POST
curl -i -X POST http://localhost:8080/inscript/modules

# Résultat 
HTTP/1.1 405
Content-Type: text/html;charset=utf-8
Content-Language: fr
Content-Length: 745
Date: Fri, 06 Nov 2026 17:27:26 GMT
<!doctype html><html lang="fr"><head><title>État HTTP 405 – Méthode non autorisée</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>État HTTP 405 – Méthode non autorisée</h1><hr class="line" /><p><b>Type</b> Rapport d'état</p><p><b>message</b> La méthode HTTP POST n&#39;est pas supportée par cette URL</p><p><b>description</b> La méthode HTTP spécifiée n'est pas autorisée pour la ressource demandée.</p><hr class="line" /><h3>Apache Tomcat/11.0.27</h3></body></html>

# Q1.1 Pourquoi une seule servlet pour /modules et /modules/*, plutôt que deux?
On utilise une seule Servlet avec les deux mappings /modules et /modules/* afin de centraliser le traitement des requêtes concernant les modules.
Cela évite de dupliquer le code dans deux Servlets différentes et facilite la maintenance de l'application.

# Q1.2 Le code du module pourrait voyager en paramètre (/modules?code=M231) ou dans le chemin(/modules/M231). Lequel convient pour désigner une ressource, lequel pour filtrer une liste?
Cette question porte sur la différence entre un paramètre d'URL (Query Parameter) et une variable dans le chemin (Path Parameter).
/modules/M231        : Désigner une ressource précise
/modules?code=M231   : Filtrer ou rechercher une liste

# Q1.3 Retirez le return qui suit sendError dans le cas de la filière inconnue. La page reçue change-t-elle ? Le code qui suit s’exécute-t-il ? Pourquoi est-ce quand même une erreur ?
Après avoir supprimé `return`, la page d'erreur reçue ne change généralement pas, car `sendError()` envoie une réponse HTTP d'erreur au client.
Cependant, le code situé après `sendError()` continue à s'exécuter, car cette méthode ne termine pas automatiquement l'exécution de la méthode Java.
Si le code suivant tente d'écrire dans la réponse, cela peut provoquer une exception si la réponse est déjà terminée ou engagée.
C'est quand même une erreur, car une filière inconnue correspond à une requête invalide. Il faut donc arrêter le traitement après l'envoi de l'erreur.
Il est important d'utiliser `return` après `sendError()` pour empêcher l'exécution du traitement normal lorsque la requête est invalide.

# Q1.4 Pourquoi le POST est-il refusé, alors que vous n’avez écrit aucune ligne pour le refuser?
Le POST est refusé parce que la classe HttpServlet fournit une implémentation par défaut de la méthode doPost() qui renvoie une erreur HTTP 405 (Method Not Allowed).
Dans notre Servlet, nous avons redéfini doGet(), mais pas doPost().
Ainsi, lorsqu'une requête POST est envoyée, c'est l'implémentation par défaut de HttpServlet qui est exécutée et qui refuse la requête.














