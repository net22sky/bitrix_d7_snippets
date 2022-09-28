
  ***

 1. [IN с логикой AND](#IN-с-логикой-AND)
 2. [Джоин по нескольким условиям](#Джоин-по-нескольким-условиям)
 3. [SELECT с подзапросом](#SELECT-с-подзапросом)
 4. [WHERE с подзапросом](#WHERE-с-подзапросом)
 5. [Джоин не пустой таблицы без дублей.](#Джоин-не-пустой-таблицы-без-дублей.)

  
  ***


## IN с логикой AND ##


Допустим нам нужно выбрать инфоблоки у которых есть элементы с CODE = ‘new’ ИЛИ CODE = ‘hot’ ИЛИ CODE = ‘exclusive’. Задача довольно легко решается передачей массива в фильтр:

```php

$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());

$query
    ->registerRuntimeField('element', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
            ],
        ]
    )
    //посчитаем количество найденых типов элементов
    //обратите внимание на DISTINCT - указывает на то что нужно считать не повторяющиеся символьные коды
    ->registerRuntimeField('COUNT_ELEMENT_CODES_VARIANTS', [
            'data_type' => 'integer',
            'expression' => ['COUNT(DISTINCT %s)', 'element.CODE'],
        ]
    )
    ->setSelect([
        'ID',
        'COUNT_ELEMENT_CODES_VARIANTS',
    ])
    ->setFilter([
        'element.CODE' => ['new', 'hot', 'exclusive']
    ]);

```

## Джоин по нескольким условиям ##


```php

$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());

$query
    //джоиним элементы с кодом hot
    ->registerRuntimeField('HOT_ELEMENT', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
                //добавим условие что нам нужны элементы с кодом hot
                '=ref.CODE' => new Bitrix\Main\DB\SqlExpression('?', 'hot'),
            ],
        ]
    )
    //считаем количество элементов с кодом hot
    ->registerRuntimeField('HOT_ELEMENTS_COUNT', [
        'data_type'=>'integer',
        'expression' => ['COUNT(%s)', 'HOT_ELEMENT.ID']
    ])
    //джоиним элементы с кодом new
    ->registerRuntimeField('NEW_ELEMENT', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
                //добавим условие что нам нужны элементы с кодом new
                '=ref.CODE' => new Bitrix\Main\DB\SqlExpression('?', 'new'),
            ],
        ]
    )
    //считаем количество элементов с кодом new
    ->registerRuntimeField('NEW_ELEMENTS_COUNT', [
        'data_type'=>'integer',
        'expression' => ['COUNT(%s)', 'NEW_ELEMENT.ID']
    ])
    //выбираем ID инфоблока, кол-во hot и new элементов
    ->setSelect([
        'ID',
        'HOT_ELEMENTS_COUNT',
        'NEW_ELEMENTS_COUNT',
    ])
    //фильтруем
    ->setFilter([
        '>HOT_ELEMENTS_COUNT' => 3,
        '>NEW_ELEMENTS_COUNT' => 5,
    ])
    //группируем выборку по ID инфоблока
    ->setGroup('ID');

```

## SELECT с подзапросом ## 

Получим количество активных элементов у инфоблоков. Для связи с родительским запросом используем Bitrix\Main\DB\SqlExpression.

```php

//формируем подзапрос - выберем только активные элементы
$subQuery = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\ElementTable::getEntity());

$subQuery
    ->registerRuntimeField('CNT', [
        'data_type' => 'integer',
        'expression' => ['COUNT(*)']
    ])
    ->setSelect([
        'CNT'
    ])
    ->setFilter([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => new Bitrix\Main\DB\SqlExpression('%s')  //сюда позже подставим алиас поля содержащий ID инфоблока из родительского запроса
    ]);
//получаем SQL подзапроса
$subQuerySql = $subQuery->getQuery();

//формируем запрос
$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());
$query
    ->registerRuntimeField('ACTIVE_ELEMENTS_CNT', [
        'expression' => ['(' . $subQuerySql . ')', 'ID'] //здесь как раз и связываем родительский запрос с подзапросом по ID инфоблока
        //не забываем обернуть в скобки SQL подзапроса, иначе запрос будет некорректным
    ])
    ->setSelect([
        'ID',
        'ACTIVE_ELEMENTS_CNT',
    ]);

```

## WHERE с подзапросом ##

Выберем инфоблоки с 10 активными элементами

```php

//формируем подзапрос
$subQuery = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\ElementTable::getEntity());
$subQuery
    ->registerRuntimeField('CNT', [
        'data_type' => 'integer',
        'expression' => ['COUNT(*)']
    ])
    ->setSelect([
        'CNT'
    ])
    ->setFilter([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => new Bitrix\Main\DB\SqlExpression('%s')
    ]);
$subQuerySql = $subQuery->getQuery();

//формируем запрос
$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());
$query
    ->registerRuntimeField('ACTIVE_ELEMENTS_CNT', [
        'expression' => ['(' . $subQuerySql . ')', 'ID']
    ])
    ->setSelect([
        'ID',
    ])
    ->setFilter([
        '=ACTIVE_ELEMENTS_CNT' => 10,
    ]);

    

 ```


 ## Джоин не пустой таблицы без дублей. ##

 Выберем инфоблоки с кодом news у которых есть хотя бы один элемент.

При этом в выборке каждая строка должна соответствовать уникальному инфоблоку, т.е. инфоблоки в выборке не должны дублироваться.

```php
$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());
$query
    ->registerRuntimeField('element', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
            ],
        ]
    )
    //регистрируем поле с минимальным ID эл-та (нам не важно какой именно этот элемент, важно есть ли в принципе минимальный ID или нет)
    ->registerRuntimeField('min_element_id', [
        'data_type'=>'integer',
        'expression' => ['MIN(%s)', 'element.ID']
    ])
    ->setSelect([
        'ID',
    ])
    //фильтруем
    ->setFilter([
        '!min_element_id' => false, 
        'CODE' => 'news'
    ])
    //группируем по ID инфоблока
    ->setGroup([
        'ID',
    ]);

```