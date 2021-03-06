Yii2 Linker Behavior
========================

This is a **NEW** version of a [**Yii2 ManyToMany Behavior**](https://github.com/voskobovich/yii2-many-to-many-behavior).  
It's the [**Yii2 ManyToMany Behavior**](https://github.com/voskobovich/yii2-many-to-many-behavior) only in **new implementation architecture**.

---

### Что это?
Это копия [**Yii2 ManyToMany Behavior**](https://github.com/voskobovich/yii2-many-to-many-behavior) но с другой архитектурой. 

В [Yii2 ManyToMany Behavior](https://github.com/voskobovich/yii2-many-to-many-behavior) некоторые жаловались, что у сохранения много-ко-многим одна логика и она не всем подходит.  
Я сделал так, что **логику** обновления связей **можно конфигурировать** без перепилки кода. 

В этой сборке реализовано три класса логики (апдейтера):
* **ManyToManyUpdater.php** - удаляет все связи и перезаписывает их заново. Его преимущество в том, что обновление происходит за 2 быстрых запроса в БД.
* **ManyToManySmartUpdater.php** - находит разницу и вносит корректировки в связи. Он управляется уже за 4 запроса но при этом не убивает существующие связи.
* **OneToManyUpdater.php** - стандартная логика работы со связью один-ко-многим.

Все это позволяет разработчикам писать свою логику обновления связей с **блекджеком и девочками** для своих проектов.  
В репо есть интерфейсы, абстрактные классы и все, что нужно для расширения. 

Сейчас проект в стадии RC и на новый функционал нужно допилить тестов.  
Но в целом тесты которые были на [Yii2 ManyToMany Behavior](https://github.com/voskobovich/yii2-many-to-many-behavior) на этой архитектуре **проходят успешно**.  
Так что уже **можно юзать**.

### Что на счет совместимости с [Yii2 ManyToMany Behavior](https://github.com/voskobovich/yii2-many-to-many-behavior)?
Изменились только параметры конфигурирования компонента.  
Нужно просто внимательно свериться с докой [README.md](https://github.com/voskobovich/yii2-linker-behavior/blob/master/README.md) и все будет хорошо.

---

This behavior makes it easy to maintain many-to-many and one-to-many relations in your ActiveRecord models.

[![License](https://poser.pugx.org/voskobovich/yii2-linker-behavior/license.svg)](https://packagist.org/packages/voskobovich/yii2-linker-behavior)
[![Latest Stable Version](https://poser.pugx.org/voskobovich/yii2-linker-behavior/v/stable.svg)](https://packagist.org/packages/voskobovich/yii2-linker-behavior)
[![Latest Unstable Version](https://poser.pugx.org/voskobovich/yii2-linker-behavior/v/unstable.svg)](https://packagist.org/packages/voskobovich/yii2-linker-behavior)
[![Total Downloads](https://poser.pugx.org/voskobovich/yii2-linker-behavior/downloads.svg)](https://packagist.org/packages/voskobovich/yii2-linker-behavior)
[![Build Status](https://travis-ci.org/voskobovich/yii2-linker-behavior.svg?branch=master)](https://travis-ci.org/voskobovich/yii2-linker-behavior)

Support
---
[GutHub issues](https://github.com/voskobovich/yii2-linker-behavior/issues).

Usage
-----
1. In your model, add the behavior and configure it
2. In your model, add validation rules for the attributes created by the behavior
3. In your view, create form fields for the attributes

Adding and configuring the behavior
-----------------------------------

As an example, let's assume you are dealing with entities like `Book`, `Author` and `Review`. The `Book` model has the following relationships:
```php
public function getAuthors()
{
    return $this->hasMany(
        Author::className(),
        ['id' => 'author_id']
    )->viaTable(
        '{{%book_has_author}}',
        ['book_id' => 'id']
    );
}

public function getReviews()
{
    return $this->hasMany(Review::className(), ['id' => 'review_id']);
}
```
In the same model, the behaviour can be configured like so:
```php
public function behaviors()
{
    return [
        [
            'class' => \voskobovich\linker\LinkerBehavior::className(),
            'relations' => [
                'author_ids' => 'authors',
				'review_ids' => 'reviews',
            ],
        ],
    ];
}
```

Relation names don't need to end in `_ids`, and you can use any name for a relation. It is recommended to use meaningful names, though.

Adding validation rules
-------------------------

The attributes are created automatically. However, you must supply a validation rule for them (usually a `safe` validator):
```php
public function rules()
{
    return [
        [['author_ids', 'review_ids'], 'each', 'rule' => ['integer']]
    ];
}
```

Creating form fields
--------------------

By default, the behavior will accept data from a multiselect field:
```php
<?= $form->field($model, 'author_ids')
    ->dropDownList($authorsAsArray, ['multiple' => true]) ?>
...
<?= $form->field($model, 'review_ids')
    ->dropDownList($reviewsAsArray, ['multiple' => true]) ?>
```

Known issues and limitations
----------------------------

* Composite primary keys are not supported.
* Junction table for many-to-many links is updated using the connection from the primary model.
* When using a function to calculate the default value, keep in mind that this function is called once, right before the relations are saved, and then its result is used to update all relevant rows using one query.
* Relations are saved using DAO (i. e. by manipulating the tables directly).

### Custom getters and setters ###

Attributes like `author_ids` and `review_ids` in the `Book` model are created automatically. By default, they are configured to accept data from a standard select input (see below). However, it is possible to use custom getter and setter functions, which may be useful for interaction with more complex frontend scripts. It is possible to define many alternative getters and setters for a given attribute:

```php
//...
'author_ids' => [
    'authors',
    'fields' => [
        'json' => [
            'get' => function($value) {
                //from internal representation (array) to user type
                return JSON::encode($value);
            },
            'set' => function($value) {
                //from user type to internal representation (array)
                return JSON::decode($value);
            },
        ],
        'string' => [
            'get' => function($value) {
                //from internal representation (array) to user type
                return implode(',', $value);
            },
            'set' => function($value) {
                //from user type to internal representation (array)
                return explode(',', $value);
            },
        ],
    ],
]
//...
```

Field name is concatenated to the attribute name with an underscore. In this example, accessing `$model->author_ids` will result in an array of IDs, `$model->author_ids_json` will return a JSON string and `$model->author_ids_string` will return a comma-separated string of IDs. Setters work similarly.

Getters and setters may be ommitted to fall back to default behavior (arrays of IDs).

###### NOTE ######
The setter function receives whatever data comes through the `$_REQUEST` and is expected to return the array of the related model IDs. The getter function receives the array of the related model IDs.

###### COMPATIBILITY NOTE ######
Specifying getters and setters for the primary attribute (`author_ids` in the above example) is still supported, but not recommended. Best practice is to use primary attribute to get and set values as array of IDs and create `fields` to use other getters and setters.

### Custom junction table values ###

For seting additional values in junction table (apart columns required for relation), you can use `viaTableAttributesValue`:

```php
...
'author_ids' => [
    'authors',
    'updater' => [
        'viaTableAttributesValue' => [
            'status_key' => BookHasAuthor::STATUS_ACTIVE,
            'created_at' => function() {
                return new \yii\db\Expression('NOW()');
            },
            'is_main' => function($updater, $relatedPk, $isNewRecord) {
                return array_search($relatedPk, $model->author_ids) === 0;
            },
        ],
    ]
]
...
```

### Setting default values for orphaned models ###

When one-to-many relations are saved, old links are removed and new links are created. To remove an old link, the corresponding foreign-key column is set to a certain value. It is `NULL` by default, but can be configured differently. Note that your database must support your chosen default value, so if you are using `NULL` as a default value, the field must be nullable.

You can supply a constant value like so:

```php
...
'review_ids' => [
    'reviews',
    'updater' => [
        'fallbackValue' => 17,
    ]
],
...
```

It is also possible to assign the default value to `NULL` explicitly, like so: `'fallbackValue' => null`. Another option is to provide a function to calculate the default value:

```php
...
'review_ids' => [
    'reviews',
    'updater' => [
        'fallbackValue' => function($model, $relationName, $attributeName) {
            //default value calculation
            //...
            return $fallbackValue;
        },
    ]
],
...
```

The function accepts 3 parameters. In our example `$model` is the instance of the `Book` class (owner of the behavior), `$relationName` is `'reviews'` and `$attributeName` is `'review_ids'`.

If you need the db connection inside this function, it is recommended to obtain it from either the primary model (`Book`) or the secondary model (`Review`).
```php
function($model, $relationName, $attributeName) {
    //get db connection from primary model (Book)
    $connection = $model::getDb();
    ...
    //OR get db connection from secondary model (Review)
    $secondaryModelClass = $model->getRelation($relationName)->modelClass;
    $connection = $secondaryModelClass::getDb();
    ...
    //further value calculation logic (db query)
```

### Applying the behaviour several times to a single relationship ###

It is possible to use this behavior for a single relationship multiple times in a single model. This is not recommended, however.

### Using the behaviour with relations that are using the same junction table ###

When you are implementing multiple ManyToMany relations in the same model, and they are using same junction table,
you may face and issue when your junction records will not be saved properly.

This happens because old junction records are dropped each time new relation is saved.
To avoid deletion of records that were just saved, you will need to set `viaTableCondition` param.

This delete condition will be merged with primary delete condition and may be used to fine tune your delete query.

For example, let's imagine that we develop a scientific database for botanical laboratory.
We have a model called "Sample" for different plants samples, model called "Attachment" for related files
(photos or documents) and junction table "sample_attachments".
And we want to divide all those files into separate fields in the "Sample" model (raw material pictures,
molecular structure, etc) by introducing field "type" in the junction table.
In such case, the resulting "Sample" model will look like this:

```php
    public function behaviors()
    {
        return [
            'manyToMany' => [
                'class' => LinkerBehavior::className(),
                'relations' => [
                    'rawMaterialPicturesList' => [
                        'rawMaterialPictures',
                        'updater' => [
                            'viaTableAttributesValue' => [
                                'type_key' => 'RAW_MATERIAL_PICTURES',
                            ],
                            'viaTableCondition' => [
                                'type_key' => 'RAW_MATERIAL_PICTURES',
                            ],
                        ]
                    ],
                    'molecularStructureList' => [
                        'molecularStructure',
                        'updater' => [
                            'viaTableAttributesValue' => [
                                'type_key' => 'MOLECULAR_STRUCTURE',
                            ],
                            'viaTableCondition' => [
                                'type_key' => 'MOLECULAR_STRUCTURE',
                            ],
                        ]
                    ],
                ],
            ],
        ];
    }
    
    public function getRawMaterialPictures()
    {
        return $this->hasMany(
            Attachment::className(),
            ['id' => 'related_id']
        )->viaTable(
            'sample_attachments',
            ['current_id' => 'id'],
            function ($query) {
                $query->andWhere([
                    'type_key' => 'RAW_MATERIAL_PICTURES',
                ]);
                return $query;
            }
        );
    }
    
    public function getMolecularStructure()
    {
        return $this->hasMany(
            Attachment::className(),
            ['id' => 'related_id']
        )->viaTable(
            'sample_attachments',
            ['current_id' => 'id'],
            function ($query) {
                $query->andWhere([
                    'type_key' => 'MOLECULAR_STRUCTURE',
                ]);
                return $query;
            }
        );
    }
    
```

Installation
------------

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require --prefer-dist voskobovich/yii2-linker-behavior "^4.0"
```

or add

```
"voskobovich/yii2-linker-behavior": "^4.0"
```

to the require section of your `composer.json` file.
