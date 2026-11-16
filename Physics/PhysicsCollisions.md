# Physique et Collisions

# Collisions Objects

Dans Godot, on trouve 4 types d’objets permettant de détecter des collisions: 

- **`Area`**- Les nœuds Area permettent de détecter lorsque d’autres objets physiques entrent à l’intérieur de leur zone de détection, mais également lorsqu’ils en sortent, et d’émettre des signaux à ces moments clés. Ces nœuds peuvent également être utilisés pour modifier les propriétés physiques dans une zone, comme la gravité par exemple.
- **`StaticBody`**- Les nœuds StaticBody permettent de participer dans les interactions de collisions mais ne sont pas bougés en réponses à celles-ci, et ne sont pas déplacés non plus par le moteur physique. Ces nœuds sont principalement utilisés pour des objets immobiles faisant partie de l’environnement, comme des murs, qui n’ont pas besoin de comportements physiques dynamiques.
- **`RigidBody`**- Les nœuds RigidBody sont des nœuds contrôlés par le moteur physique. Afin de les manipuler, on va y appliquer des forces afin que le moteur physique puisse calculer et appliquer les mouvements résultant des forces appliquées.
- **`CharacterBody`**- Les nœuds CharacterBody sont des nœuds qui permettent d’appliquer une détection des collisions, mais sans application des comportements dictés par le moteur physique, ainsi les mouvements et les réponses aux collisions doivent être fait à la main. Malgré son nom, ce type de nœuds n’est pas utilisé uniquement pour des personnages mais également pour tous les objets physiques dont le comportement devrait être contrôlé de manière précise ou spécifique (comme par exemple des projectiles à tête chercheuse, des véhicules, etc…).

Afin de fonctionner, ces nœuds ont également besoin d’au moins un nœud CollisionShape afin d’avoir une zone de collision.

# Collisions Layers et Collisions Masks

Les layers de collisions sont un système permettant de gérer les interactions entre plusieurs objets physiques organisés dans des groupes.

Tous les Collisions Objects ont deux propriétés, qu’on peut retrouver dans l’inspector: 

- un **collision layer** - le ou les groupes auxquels cet objet appartient (tous les objets sont dans le layer 1 par défaut).
- un **collision mask** - le ou les groupes que l’objet peut détecter (tous les objets détectent le layer 1 par défaut).

C’est à dire que si un Collision Object appartenant au Layer 2 entre dans la zone de collision d’une Area, il ne sera détecté par celle-ci (et les signaux ne seront émis) que si ce Collision Object que s’il appartient au Layer 2 dans son collision layer. 

Afin d’avoir une utilisation plus claire des Layers, il est possible de les nommer en allant dans **`Project Settings > Layer Names > 2D Physics`** ou **`Project Settings > Layer Names > 3D Physics`** ou en appuyant sur les ****trois petits points à droite des layer collisions ou des layer mask d’un Collision Object et d’appuyer sur `Edit Layer Names` .

# Detection :

## Area

Une Area va pouvoir détecter les Collisions Object qui entrent et sortent de sa CollisionShape via divers signaux :

- `area_entered(Area2D/3D)` et `area_exited(Area2D/3D)` lorsqu’une autre Area entre ou sort de sa CollisionShape.
- `body_entered(Area2D/3D)` et `body_exited(Area2D/3D)` lorsqu’un objet héritant de PhysicsBody entre ou sort de sa CollisionShape.

> Les nœuds `StaticBody` ,`RigidBody`et `CharacterBody`sont tous les trois des classes héritants de PhysicsBody.
> 

## RigidBody

Par défaut, les RigidBody ne détectent pas les collisions, pour activer la détection des `PhysicsBody` , il faut mettre la valeur de `contact monitor = true` et mettre la valeur de `max contact reported` à une valeur supérieure à 0 (attention, le Rigidbody ne pourra pas détecter plus de collisions simultanée que la valeur que vous avez choisi ici!). 

Vous pourrez ensuite détecter les collisions via les signaux : 

- `body_entered(Area2D/3D)` et `body_exited(Area2D/3D)` lorsqu’un objet héritant de PhysicsBody entre en collision ou quitte la collision avec le RigidBody.

## CharacterBody

Dans le cas du CharacterBody, celui-ci détecte les collisions établies par le moteur physique mais il n’émet pas de signal et n’a pas de comportement particulier lorsqu’il entre en collision avec un autre Collision Object. Cependant, il possède des méthodes nous permettent de déplacer et de récupérer les informations de collision.

`move_and_slide()` est la méthode qui va nous permettre d’appliquer un déplacement au CharacterBody, si celui-ci entre en collision avec un autre objet physique, il va glisser le long de la collision plutôt que se stopper net.

> Dans la plupart des cas, si on veut déplacer un Character Body, il est toujours plus recommandé de le faire dans la fonction `physics_process` en appelant la méthode `move_and_slide()` en utilisant la vélocité du CharacterBody puisque c’est grâce à cette valeur que la méthode `move_and_slide()` va calculer quel déplacement appliquer.
> 

Afin de récupérer les informations de collisions, on va préférer utiliser la méthode `move_and_collide(velocity)` qui prend en paramètre une vélocité, donc le déplacement que l’on va effectuer (n’oubliez pas de multiplier votre vélocité par le delta de votre process pour que le mouvement soit indépendant du framerate ou du taux de rafraichissement du moteur physique), et cette fonction va nous retourner un objet de type [KinematicCollision2D](https://docs.godotengine.org/en/stable/classes/class_kinematiccollision2d.html#class-kinematiccollision2d), qui contiendra des informations importantes sur la collision comme le point où la collision a eu lieu, la normal de la face où il y a eu la collision ou encore l’objet avec lequel notre CharacterBody est entré en collision.

![Images/image.png](image.png)

Voici un exemple dans lequel on récupère la position de la collision après avoir utilisé la méthode `move_and_collide()` .

## Solutions alternatives

Dans le cas des RigidBody et des CharacterBody, on a d’autres manières de récupérer les informations de collisions: 

Dans le cas du RigidBody, on peut utiliser la fonction`get_contact()` afin de récupérer les informations de collisions à un instant T 

- Attention à bien vérifier que `get_contact()` puisse nous retourner des informations en utilisant la fonction `get_contact_count()` pour vérifier qu’il y a au moins un collision.

Dans le cas du CharacterBody, on peut utiliser la fonction `get_slide_collision()` afin de récupérer les informations de collisions depuis le dernier appel de `move_and_slide()` .

- Attention à bien vérifier que `get_slide_collision()` puisse nous retourner des informations en utilisant la fonction `get_slide_collision_count()` pour vérifier qu’il y a au moins un collision.