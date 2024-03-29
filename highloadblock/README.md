# API D7 работа с Highload


***
1. [Подключаем модуль для работы с highloadblock блоками]()
2. [Новый способ в ядре d7]()
3. [Получение объекта сущности хайлоадблока (Entity Data Class) по ID хайлоадблока:](#3)
4. [Получение ID хайлоадблока по названию сущности:](#4)
***


## Подключаем модуль для работы с highloadblock блоками

Старый способ

```php
CModule::IncludeModule("highloadblock");

```

## Новый способ в ядре d7

```php

use Bitrix\Main\Loader;

Loader::includeModule("highloadblock");

```

## <a name=3>Получение объекта сущности хайлоадблока (Entity Data Class) по ID хайлоадблока:</a>

```php
$hlblock = \Bitrix\Highloadblock\HighloadBlockTable::getById($HLBLOCK_ID)->fetch();

$entity = \Bitrix\Highloadblock\HighloadBlockTable::compileEntity($hlblock);

$entityDataClass = $entity->getDataClass();
```

## <a name=4>Получение ID хайлоадблока по названию сущности:</a>

```php
$result = \Bitrix\Highloadblock\HighloadBlockTable::getList(array('filter'=>array('=NAME'=>"MyEntityName")));

if($row = $result->fetch())

{

    $HLBLOCK_ID = $row["ID"];

}
```

## <a name=5>Делаем выборку из highloadblock блока метод getlist</A>

```php

use Bitrix\Main\Loader;

Loader::includeModule("highloadblock");

use Bitrix\Highloadblock as HL;
use Bitrix\Main\Entity;

$hlbl = 1; // Указываем ID нашего highloadblock блока к которому будет делать запросы.
$hlblock = HL\HighloadBlockTable::getById($hlbl)->fetch();

$entity = HL\HighloadBlockTable::compileEntity($hlblock);
$entity_data_class = $entity->getDataClass();

$rsData = $entity_data_class::getList(array(
   "select" => array("*"),
   "order" => array("ID" => "ASC"),
   "filter" => array("UF_PRODUCT_ID"=>"77","UF_TYPE"=>'33')  // Задаем параметры фильтра выборки
));

while($arData = $rsData->Fetch()){
   var_dump($arData);
}
```

## <a name=6>Пример добавления записи в highloadblock</a>

```php

use Bitrix\Main\Loader;

Loader::includeModule("highloadblock");

use Bitrix\Highloadblock as HL;
use Bitrix\Main\Entity;

$hlbl = 1; // Указываем ID нашего highloadblock блока к которому будет делать запросы.
$hlblock = HL\HighloadBlockTable::getById($hlbl)->fetch();

$entity = HL\HighloadBlockTable::compileEntity($hlblock);
$entity_data_class = $entity->getDataClass();

   // Массив полей для добавления
   $data = array(
      "UF_TYPE"=>'33',
      "UF_COUNT"=>'1',
      "UF_DATA"=>date("d.m.Y")
   );

   $result = $entity_data_class::add($data);
```

## <a name=7>Пример обновления записи по ID в highloadblock</a>

```php
use Bitrix\Main\Loader;

Loader::includeModule("highloadblock");

use Bitrix\Highloadblock as HL;
use Bitrix\Main\Entity;

$hlbl = 1; // Указываем ID нашего highloadblock блока к которому будет делать запросы.
$hlblock = HL\HighloadBlockTable::getById($hlbl)->fetch();

$entity = HL\HighloadBlockTable::compileEntity($hlblock);
$entity_data_class = $entity->getDataClass();


   // Массив полей для обновления
   $data = array(
      "UF_TYPE"=>'33',
      "UF_COUNT"=>'1',
      "UF_DATA"=>date("d.m.Y")
   );

   $result = $entity_data_class::update("77", $data); // где 77 -  id обновляемой записи

```

## <a name=8>Пример удаление записи по ID из highloadblock</a>

```php

use Bitrix\Main\Loader;

Loader::includeModule("highloadblock");

use Bitrix\Highloadblock as HL;
use Bitrix\Main\Entity;

$hlbl = 1; // Указываем ID нашего highloadblock блока к которому будет делать запросы.
$hlblock = HL\HighloadBlockTable::getById($hlbl)->fetch();

$entity = HL\HighloadBlockTable::compileEntity($hlblock);
$entity_data_class = $entity->getDataClass();

$entity_data_class::Delete("77");  // где 77 -  id удаляемой записи

```

## <a name=9>Выборка данных по фильтру:</a>

```php
$result = $entityDataClass::getList(array(

    "select" => array("*"),

    "order" => array("ID"=>"DESC"),

    "filter" => Array("UF_USER_ID"=>4),

));

while ($arRow = $result->Fetch())

{

    print_r($arRow);

}
```

## <a name=10>Выборка данных по фильтру с указанием максимального количества:</a>

```php
$result = $entityDataClass::getList(array(

    "select" => array("ID","UF_URL"),

    "order" => array("UF_USER_ID"=>"ASC"),

    "filter" => Array("UF_USER_ID"=>4),

    "limit" => 5,

));

while ($arRow = $result->Fetch())

{

    print_r($arRow);

}
```

## <a name=11>Получение количества элементов по фильтру:</a>

```php
$result = $entityDataClass::getList(array(

    'select' => array('CNT'),

    'filter' => array('UF_ACTIVE'=>true),

    'runtime' => array(

        new \Bitrix\Main\Entity\ExpressionField('CNT', 'COUNT(*)'),

    ),

));

if($arRow = $result->Fetch())

    $count = $arRow['CNT'];

```

## <a name=12>Использование ShortHand-функции для выборки элемента с идентификатором из переменной $ID:</a>

```php
$result = $entityDataClass::getById($ID);



if($arRow = $result->Fetch())

{

    print_r($arRow);

}
```

## <a name=13>Добавление элемента хайлоадблока:</a>

```php
$arFields = array (

    'UF_USER_ID' => $userID,

);



$result = $entityDataClass::add($arFields);



if($result->isSuccess())

{

    $ID = $result->getId(); // получаем ID созданного элемента хайлоадблока

}

else

{

    $errors = $result->getErrorMessages(); // получаем сообщения об ошибках

    /* $errors будет иметь следующий формат:

    Array

    (

        [0] => The value of "Date" is not a valid date or time.<br>

    )

    */

}
```

## <a name=14>Обновление элемента хайлоадблока с идентификатором, содержащимся в переменной $ID:</a>

```php

$arFields = array (

    'UF_USER_ID' => $userID,

);



$result = $entityDataClass::update($ID,$arFields);



if(!$result->isSuccess())

    $errors = $result->getErrorMessages(); // получаем сообщения об ошибках
```

## <a name=15>Удаление элемента хайлоадблока с идентификатором, содержащимся в переменной $ID:</a>

```php
$result = $entity_data_class::delete($ID);



if(!$result->isSuccess())

    $errors = $result->getErrorMessages(); // получаем сообщения об ошибках
```
