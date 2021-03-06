# Data Augmentation ou l'art de ne jamais overfitter votre dataset

Les réseaux de neurones par convolution (CNN) sont des systèmes qui permettent d'extraire des informations facilement sur des objets graphiques comme ils permettent de synthétiser les données spatiales rapidement. Cependant, comme ils font également partie de la famille des algorithmes de Machine Learning, ils sont soumis aux mêmes problèmes à savoir l'underfitting et l'overfitting. Pour rappel, l'underfitting survient lorsque la complexité d'un modèle est bien supérieure à la quantité d'information contenue dans le jeu de données (dataset). L'optimisation du modèle ne converge pas vers un extremum et le modèle décrit modérément les motifs contenus dans le jeu de données (biais). A l'inverse, l'overfitting intervient lorsque la complexité du modèle est inférieure à la quantité d'information contenue dans le jeu de données. Le modèle a alors tendance à se focaliser sur certains motifs du jeu d'entraînement qui pourraient ne pas être représentatifs dans le cas d'un dataset plus large. Les prédictions du modèle présentent de la	variance et le modèle a du mal à généraliser pour d'autres données en entrée. 

La performance d'un CNN peut être décrite par sa capacité à classifier ou détecter des objets dans des situations différentes, que l'on appelle l'invariance d'un CNN. Plus précisément, un CNN peut être invariant en translation, rotation, taille, illumination, etc ... L'augmentation de données vise dans le cas des CNN à repousser le moment où survient l'overfitting. Elle consiste à augmenter le panel de conditions différentes pour les images du dataset en effectuant des transformations mineures sur les images déjà présentes dans le dataset. On entraîne donc le CNN avec un dataset augmenté artificiellement.

Dans le pipeline d'opérations visant à générer un classificateur ou un détecteur à partir du CNN, deux méthodes existent pour réaliser l'entraînement :

- L'augmentation hors-ligne (*offline augmentation*) : Toutes les transformations sur les images sont réalisées avant l'entraînement. Généralement, les transformations concernent toutes les images ce qui augmente considérablement la taille du dataset. 
- L'augmentation à la volée (*online/on the fly augmentation*) : Les transformations sont réalisées seulement sur un sous-ensemble du jeu de données, puis le dataset résultant est donné comme entrée au CNN durant l'entraînement. Cette méthode est mieux adaptée pour des datasets larges comme elle permet de ne pas augmenter trop sa taille qui est déjà grande. 

### Les différentes transformations pour une augmentation de données

Quelques transformations sont expliquées ci-dessous. Pour les transformations qui induisent une image finale qui ne remplirait pas entièrement le cadre d'origine, des stratégies de complétion seront évoquées plus tard. 

- **Symétrie** qui peut être horizontale ou verticale. Différentes fonctions existent de la forme `tf.image.flip_{}` ou encore `tf.image.random_flip_{}` selon la transformation à réaliser. 

- **Rotation** qui peut ou non préserver la taille de l'image d'origine. Pour une série de rotations de &pm; 90°, `tf.image.rot90(img, k=1)`peut être utilisé où `k` représente le nombre de rotations de 90° à effectuer. Pour des rotations avec angles plus particulier, `tf.contrib.image.rotate(y, angles=3.1415)` peut être utilisé où l'angle `angles` est en radians. Il existe aussi une fonction du package Scikit-Learn pour effectuer des rotations particulières : `skimage.transform.rotate(img, angle=45, mode='reflect')`. L'angle `angle` est bien en degrés et l'attribut `mode` renseigne sur le mode de remplissage de l'espace vide (qui sera expliqué plus tard).

- **Echelle** pour effectuer un zoom ou dézoom. Dans le cas d'un zoom, il faut recadrer l'image à taille originale. Dans le cas d'un dézoom, il faut choisir une stratégie pour remplir l'espace vide créé autour de la nouvelle image. La bibliothèque Scikit-Learn fournit la fonction `skimage.transform.rescale(img, scale=2.0, mode='constant')` pour cette opération avec l'attribut `scale` > 1 dans le cas d'un zoom. L'attribut `mode` fait aussi référence à la stratégie de remplissage dans le cas d'un dézoom. 

- **Recadrage**  qui est différent du facteur d'échelle. Ici, on sélectione arbitrairement une partie de l'image d'origine que l'on recadre à la taille d'origine. Ces deux opérations sont effectuées successivement via deux fonctions de la biliothèque Tensorflow `tf.random_crop(x, size = crop_size, seed = seed)` et `tf.images.resize_images(x, size = original_size)` où `crop_size` et `original_size` sont les deux vecteurs tri-dimensionnels de la forme [height, width, channels] contenant les tailles d'image d'origine et à recadrer. 

- **Translation** la plus courante transformation d'augmentation de données. En quelque sorte, elle force le CNN à "regarder" de partout. Cette transformation est effectuée en deux temps par Tensorflow. La première fonction `tf.image.pad_to_bounding_box` permet d'ajouter un certain nombre de colonnes,lignes de 0 autour de l'image. Ensuite la fonction `tf.image.crop_to_bounding_box` recadre l'image selon la taille de départ à partir d'un côté spécifié. Le réglage des offsets permet de définir le déplacement de l'image.

- **Bruit Gaussien** Le sur-apprentissage (overfitting) survient lorsque le modèle essaie d'apprendre des motifs qui reviennent avec une fréquence élevée dans les données. Ils ne sont pas intéressants comme ils pourraient ne pas être représentatifs d'un dataset plus large. Une méthode d'augmentation supplémentaire consiste à utiliser une gaussienne en fréquence, autrement appelée un bruit gaussien qui a des composantes selon toutes les fréquences et une valeur moyenne de 0. Ainsi, le bruit va déranger les fréquences hautes pour que le réseau de neurones ne se focalise pas dessus. Cependant, les autres fréquences sont aussi dérangées, mais le réseau devrait plus facilement s'en accommoder. On commence par créer le bruit de la même taille que l'objet à altérer avec la fonction `noise = tf.random_normal(shape, mean=0.0, stddev=1.0, dtype)` puis on l'ajoute à l'objet avec `output = tf.add(x, noise)`.
 
Technique | Résultat
:---: | :---:
Symétrie | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_flip.jpeg)
Rotation | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_rotate.jpeg)
Echelle | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_scale.jpeg)
Recadrage | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_crop.jpeg)
Translation | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_translation.jpeg)
Bruit Gaussien | ![](https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_gaussian.png)



### Les méthodes de remplissage

Pour les techniques précédentes qui déforment l'image et induisent un espace entre le cadre et les pixels de l'image, il faut convenir d'une stratégie  

Toutes les techniques que nous avons traitées sont utilisables pour la majorité des images. Seulement, dans certains cas certaines n'auront pas vraiment de sens suivant le besoin visé. En effet, une symétrie horizontale sur une image de montgolfière n'aurait pas trop de sens. Personne ne s'attend à voir un jour de montgolfière la tête en bas !

- **Constant** Les parties vides de l'image sont simplement remplies avec une couleur unie. Cela peut être particulièrement intéressant dans le cas d'une image à fond uni où l'augmentation de donnée et le remplissage peuvent être réalisés très rapidement. 

- **Contour** Les valeurs de pixels sur les bords de l'image sont propagés orthogonalement jusqu'aux limites de l'image. 

- **Réflection** Une symétrie est effectuée le long de la bordure de l'image jusqu'à remplir l'espace vide. Cette transformation peut être utile pour les paysages naturels qui contiennent des espaces relativement homogènes. 

- **Symétrie** Cette transformation est similaire à la réflection mais les pixels de bordure sont également répliqués. 

- **Replication** L'espace vide est comblé comme si l'image était répliquée au-delà du coin concerné. Cette méthode est peu utilisée comme on retrouve peu ce cas de figure dans les jeux de données.

<center><img src="https://github.com/vintel38/Object-Detection/blob/master/Content/images/DA_fill.jpeg" ...></center>
<center> De la droite, nous avons les modes de remplissage constant, contour, réflection, symétrie et réplication.
</center>

Pour conclure, ces méthodes d'augmentation de données et de remplissage sont très utiles pour ajouter des données dans un dataset sans pour autant en collecter de nouvelles. Selon le problème de détection à résoudre, une de ces méthodes sera plus performante sur les autres mais rien ne vous empêche de créer vous-même votre méthode d'augmentation et de remplissage. On peut citer également la plateforme [Roboflow](https://roboflow.com/) qui gère parfaitement l'augmentation de données en transformant aussi les étiquettes yolo dans le processus. Voilà vous en savez maintenant un peu plus sur l'augmentation de données !

### Références 

- Data Augmentation | How to use Deep Learning when you have Limited Data — Part 2, Arun Gandhi, [ nanonets.com](https://nanonets.com/blog/data-augmentation-how-to-use-deep-learning-when-you-have-limited-data-part-2/)
- Shorten, C., Khoshgoftaar, T.M. A survey on Image Data Augmentation for Deep Learning. J Big Data 6, 60 (2019). [Springer](https://journalofbigdata.springeropen.com/articles/10.1186/s40537-019-0197-0)
- Automating Data Augmentation: Practice, Theory and New Direction, [AI Stanford Lab Blog](https://ai.stanford.edu/blog/data-augmentation/), Sharon Y. Li, April 24, 2020
