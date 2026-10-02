Question 1 

Testcontainers est une bibliothèque Java qui lance de vrais conteneurs Docker pendant l'exécution des tests, puis les 
supprime à la fin. Ici, elle démarre une base PostgreSQL temporaire à laquelle l'application se connecte pendant les 
tests d'intégration. On teste ainsi avec une vraie base, identique à celle de la production, sans rien installer à la 
main, et sans que les tests dépendent d'une base partagée : chaque exécution repart d'un environnement propre et 
reproductible, en local comme dans le pipeline CI.