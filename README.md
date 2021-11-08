# Сниппеты Битрикс d7
  
  ***

 1. [Get site ID SITE_ID](#Get-site-ID-SITE_ID)
 2. [Сайт URL](#URL)
 3. [Выборка разделов инфоблока](#Выборка-разделов-инфоблока)
 4. [Работа с изображением](#Работа-с-изображением)
 5. [Работа с элементом как с объектом](Работа-с-элементом-как-с-объектом)
 6. [Свойство инфоблока типа Список](Свойство-инфоблока-типа-Список)
 7. [Данные текущего пользователя](Данные-текущего-пользователя)
 8. [Свойства элемента](Свойства-элемента)
  
  ***
### подключить модуль iblock:
```php
\Bitrix\Main\Loader::includeModule('iblock');

```
## Get site ID SITE_ID

```php
<?php
$siteId = \Bitrix\Main\SiteTable::getList([
	'select' => ['LID'],
	'limit' => 1,
	'cache' => ['ttl' => 8460000]
])->fetch()['LID'];
if($siteId) {
  
}
```

## URL   


```php
<?php
function getServerName()
{
  $site = Bitrix\Main\SiteTable::getList([
  				"select" => ["SERVER_NAME"],
  				"filter" => ["=DEF" => "Y"]
  			])->fetch();
  
  if ($site && isset($site["SERVER_NAME"]) && !empty($site["SERVER_NAME"]))
  {
  	return $site["SERVER_NAME"];
  }
  elseif (Bitrix\Main\Config\Option::get("main", "server_name"))
  {
  	return Bitrix\Main\Config\Option::get("main", "server_name");
  }
  return "";
}

$siteUrl = (\Bitrix\Main\Context::getCurrent()->getRequest()->isHttps() ? "https://" : "http://").getServerName();


```

## Выборка разделов инфоблока


```php
\Bitrix\Main\Loader::includeModule('iblock');

$rsSection = \Bitrix\Iblock\SectionTable::getList(array(
    'filter' => array(
        'IBLOCK_ID' => $IBLOCK_ID,
        'DEPTH_LEVEL' => 1,
    ), 
    'select' =>  array('ID','CODE','NAME'),
));

while ($arSection=$rsSection->fetch()) 

{
    print_r($arSection);
}


```

## Работа с файловой системой

```php

<?php
use Bitrix\Main\IO,
    Bitrix\Main\Application;
    
/** IO\File */

$file = new IO\File(Application::getDocumentRoot() . "/file.txt");

// Информация о файле
$isExist = $file->isExists(); // true, если файл существует

$dir = $file->getDirectory(); // Директория файла в виде объекта IO\Directory
$dir = $file->getDirectoryName(); // Директория файла

$fileName = $file->getName(); // Имя файла
$fileExt = $file->getExtension(); // Расширение файла
$fileSize = $file->getSize(); // Размер файла в байтах
$contentType = $file->getContentType(); // Content-type

$createdAt = $file->getCreationTime(); // Дата создания, timestamp
$accessAt = $file->getLastAccessTime(); // Дата последнего доступа, timestamp
$modifiedAt = $file->getModificationTime(); // Дата модификации, timestamp

$perms = $file->getPermissions(); // Права на файл в виде десятичного числа
$perms = substr(sprintf('%o', $file->getPermissions()), -3); // Права на файл в виде восьмеричного числа

// Действия над файлами
$content = $file->getContents(); // Получить содержание файла
$file->putContents("data"); // Записать содержимое в файл с заменой
$file->putContents("data", IO\File::APPEND); // Дописать содержимое в конец файла
$file->readFile(); // Вывести содержимое файла

$file->rename(Application::getDocumentRoot() . "/new_file.txt"); // Переместить/переименовать файл
$file->delete(); // Удалить файл

// статические варианты
$path = Application::getDocumentRoot() . "/another_file.txt";
IO\File::isFileExists($path); // Проверить существование файла

IO\File::getFileContents($path); // Получить содержание файла
IO\File::putFileContents($path, "data"); // Записать содержимое в файл с заменой
IO\File::putFileContents($path, "data", self::APPEND); // Дописать содержимое в конец файла

IO\File::deleteFile($path); // Удалить файл

/** IO\Directory */

$dir = new IO\Directory(Application::getDocumentRoot() . "/test/");
$dir->create(); // Создаёт директорию с указанным в конструкторе путём

// Информация о директории
$isExist = $dir->isExists(); // true, если директория существует

$createdAt = $dir->getCreationTime(); // Дата создания, timestamp
$accessAt = $dir->getLastAccessTime(); // Дата последнего доступа, timestamp
$modifiedAt = $dir->getModificationTime(); // Дата модификации, timestamp

$perms = $dir->getPermissions(); // Права на директорию в виде десятичного числа
$perms = substr(sprintf('%o', $dir->getPermissions()), -3); // Права на директорию в виде восьмеричного числа

// Действия над директориями
$childDir = $dir->createSubdirectory("child"); // Создает и возвращает вложенную директорию с указанным именем 
$dir->rename(Application::getDocumentRoot() . "/another_path/"); // Переместить/переименовать директорию
$dir->delete(); // Удалить директорию

// Получить массив файлов в директории
$files = $dir->getChildren(); // Массив объектов IO\File

// статические варианты
$path = Application::getDocumentRoot() . "/another_dir/";
IO\Directory::createDirectory($path);   // Создать директорию
IO\Directory::deleteDirectory($path);   // Удалить директорию

IO\Directory::isDirectoryExists($path); // Проверить существование

/** IO\Path */

$path = Application::getDocumentRoot() . "/some_dir/some_file.ext";
$fileExt = IO\Path::getExtension($path); // Возвращает расширение файла
$fileName = IO\Path::getName($path);     // Возвращает имя файла
$fileDir = IO\Path::getDirectory($path); // Возвращает директорию файла (полный путь)

```

## Работа с датой и временем

```php

<?php
use Bitrix\Main\Type\DateTime;

// Объект Datetime является практически аналогом встроенного в PHP класса \DateTime, но от него не наследуется. 
// Объект Битрикс-класса можно получить из объекта PHP-класса с помощью метода createFromPhp или из timestamp с помощью createFromTimestamp
$objDateTime = DateTime::createFromPhp(new \DateTime('2000-01-01'));
$objDateTime = DateTime::createFromTimestamp(1346506620);

// Также есть конструкторы
// Текущее время:
$objDateTime = new DateTime();
// Из строки в формате текущего сайта
$objDateTime = new DateTime("25.12.2012 12:30:00");
// Из строки с указанием формата:
$objDateTime = new DateTime("2007-05-14 12:10:00", "Y-m-d H:i:s");

// Из объекта можно получить представление в виде timestamp
$objDateTime->getTimestamp();
// в виде строки в формате текущего сайта
$objDateTime->toString();
// в произвольном формате (фактически обёртка над DateTime::format)
$objDateTime->format("Y-m-d H:i:s");

// Метод add реализует сложение и вычитание дат, 
// можно указывать смещение словами years, months, days, weeks, hours, minutes, seconds и знаками +/-
// Метод add изменяет объект
$objDateTime = new DateTime("01.01.2012 00:00:00"); // "2012-01-01 00:00:00"
$objDateTime->add("1 day"); // "2012-01-01 00:00:00" => "2012-01-02 00:00:00"
$objDateTime->add("-1 day"); // "2012-01-01 00:00:00" =>"2011-12-31 00:00:00"
$objDateTime->add("3 months - 5 days + 10 minutes"); // "2012-01-01 00:00:00" =>"2012-03-27 00:10:00"

// Также в add можно указывать смещение в формате DateInterval (но буква P в начале строки необязательна)
$objDateTime = new DateTime("01.01.2012 00:00:00"); // "2012-01-01 00:00:00"
$objDateTime->add("7M5DT2M"); // "2012-01-01 00:00:00" =>"2012-08-06 00:02:00"
$objDateTime->add("-2YT10M"); // "2012-01-01 00:00:00" =>"2009-12-31 23:50:00"

// При указании смещения следует указывать периоды, от большей единицы до меньшей (например, года, месяцы, часы)

```

## Работа с изображением

### Получаем полный путь

```php
CFile::GetPath(315);

```
### Resize

```php
//Мы можем использовать следующие константы преобразований:
//BX_RESIZE_IMAGE_EXACT — масштабирует в прямоугольник $arSize c сохранением пропорций, обрезая лишнее;
//BX_RESIZE_IMAGE_PROPORTIONAL — масштабирует с сохранением пропорций, размер ограничивается $arSize;
//BX_RESIZE_IMAGE_PROPORTIONAL_ALT — масштабирует с сохранением пропорций, размер ограничивается $arSize, улучшенная обработка вертикальных картинок.

$arFileTmp = CFile::ResizeImageGet(
	$row['PREVIEW_PICTURE'],
	array("width" => 1000, "height" => 1000),
	BX_RESIZE_IMAGE_PROPORTIONAL,
	true
);
echo $arFileTmp["src"];
```
### Работа с элементом как с объектом
#### для одного элемента:
```php
$element = \Bitrix\Iblock\Elements\ElementCatalogTable::getByPrimary($elementId, array(
    'select' => array('ID', 'NAME', 'DETAIL_PICTURE')
))->fetchObject();

```
#### для нескольких элементов:
```php
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NAME', 'DETAIL_PICTURE'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchCollection();
```

### Свойство инфоблока типа Список
```php
<?php
// Выбрать все варианты значений свойства типа Список с идентификатором 10
$rsEnum = \Bitrix\Iblock\PropertyEnumerationTable::getList([
	'filter' => ['PROPERTY_ID' => 10],
	'select' => ['ID'],
	'cache' =>  ['ttl', 2678400],
]);
while($arEnum = $rsEnum->fetch())
{
  print_r($arEnum);
}


```

### Данные текущего пользователя

```php
<?php
// ID текущего авторизованного пользователя
$userId = \Bitrix\Main\Engine\CurrentUser::get()->getId();
// авторизован ли пользователь
$isAuth = (bool)\Bitrix\Main\Engine\CurrentUser::get()->getId();
// админ?
$isAdmin = \Bitrix\Main\Engine\CurrentUser::get()->isAdmin();
// логин текущего пользователя
$login = \Bitrix\Main\Engine\CurrentUser::get()->getLogin();
// массив групп текущего пользователя
$arUserGroups = \Bitrix\Main\Engine\CurrentUser::get()->getUserGroups();
// имя, фамилию и пр. текущего пользователя, отформатированное в соответствии с настройками
$userName = \Bitrix\Main\Engine\CurrentUser::get()->getFormattedName();
// полное имя текущего пользователя
$fullUserName = \Bitrix\Main\Engine\CurrentUser::get()->getFullName();
// имя текущего пользователя
$userName = \Bitrix\Main\Engine\CurrentUser::get()->getName();
// фамилию текущего пользователя
$lastName = \Bitrix\Main\Engine\CurrentUser::get()->getLastName();
// отчество текущего пользователя
$secondName = \Bitrix\Main\Engine\CurrentUser::get()->getSecondName();
// может ли текущий пользователь выполнить операцию с кодом операции в переменной $operationName
$canDoOperation = \Bitrix\Main\Engine\CurrentUser::get()->canDoOperation($operationName);


```
### Свойства элемента

```php
<?php
// Свойства элементов (когда мы работаем с ними в виде объекта) получаются при помощи геттеров 
// (методов getXXXX где XXXX — код свойства записанный как CamelCase). 
// У каждого свойства есть поле значения VALUE и описания DESCRIPTION 
// и соответcтвующие методы для доступа к ним getValue() и getDescription().

$product = \Bitrix\Iblock\Elements\ElementProductsTable::getByPrimary(10, [ //10 - ID товара
    'select' => ['ID', 'NAME', 'PREVIEW_TEXT', 'DETAIL_PICTURE', 'MANUFACTURER', 'MATERIAL'], //MANUFACTURER и MATERIAL свойства типа «Строка»
])->fetchObject();

// свойства типа Текст или Число
$product->getManufacturer()->getValue();

// Получить значения коллекции множественного свойства типа Справочник
$elements = \Bitrix\Iblock\Elements\ElementCatalogTable::getList([
    'select' => ['ID', 'NAME', 'DETAIL_PICTURE', 'BRAND_REF'],
    'filter' => [
        'ID' => $elementId,
    ],
])->fetchCollection();
foreach ($elements as $element) {
    foreach ($element->getBrandRef()->getAll() as $value) {
        var_dump($value->getValue()); // string(6) "brand1"
    }
}

// Чтобы получить дополнительную информацию для некоторых типов свойств,
// нужно указать дополнительный ключ при выборке свойства:
// FILE — свойство типа файл
// ITEM — свойство типа список,
// ELEMENT — свойство типа привязка к элементу инфоблока
// SECTION — свойство типа привязка к разделу инфоблока
// Например, у элемента есть следующие свойства типов:
// MORE_PHOTO — Картинки галереи (тип файл)
// NEWPRODUCT — Новинка (тип список)
// RECOMMEND — С этим товаром рекомендуем (тип привязка к элементу)
// NEWS_SECTION — Показывать в рекламном блоке в новостях (тип привязка к разделу)
$product = \Bitrix\Iblock\Elements\ElementProductsTable::getByPrimary(10, [
    'select' => [
        'ID',
        'NAME',
        'PREVIEW_TEXT',
        'DETAIL_PICTURE',
        'MORE_PHOTO.FILE',
        'NEWPRODUCT.ITEM',
        'RECOMMEND.ELEMENT',
        'NEWS_SECTION.SECTION'
    ],
])->fetchObject();

// Свойство типа файл
//Выведем доп.фотографии товара
foreach ($product->getMorePhoto()->getAll() as $photo){
    echo '<img src="/upload/' . $photo->getFile()->getSubdir() . '/' . $photo->getFile()->getFileName() . '" alt="'. $product->getName() .'" />';
}

// Свойство типа список
$product->getNewproduct()->getItem()->getId();
//int(1)
$product->getNewproduct()->getItem()->getXmlId();
//string(1) "Y"
$product->getNewproduct()->getItem()->getValue();
//string(4) "да"

// Свойство типа привязка к элементам инфоблока
//Получим привязанные элементы
foreach ($product->getRecommend()->getAll() as $recommendedProduct){
    echo 'Рекомендуемый товар ID -' . $recommendedProduct->getElement()->getId() . ' наименование - ' . $recommendedProduct->getElement()->getName() . '<br/>';
}

// Свойство типа привязка к разделу инфоблока
//Получим id привязанного раздела
$product->getNewsSection()->getSection()->getId();
//Получим наименование привязанного раздела
$product->getNewsSection()->getSection()->getName();

```

### Работа с базой данных

```php
<?php
// Получить соединение с основной базой данных
$connection = \Bitrix\Main\Application::getConnection();

// Подключение к дополнительной базе данных
$connection = \Bitrix\Main\Application::getConnection(DB_NAME);
$result = $connection->query('SELECT * FROM table_name ORDER BY id DESC');

// dropTable
if ($connection->isTableExists(TABLE_NAME))
	$connection->dropTable(TABLE_NAME);

// create table
$sql = 'CREATE TABLE IF NOT EXISTS `'.TABLE_NAME.'` ('.implode(',', $arColumns).');';
$connection->queryExecute($sql);

// Запрос без получения результата
$sql = "INSERT INTO ".TABLE_NAME."(".$insert[0].") "."VALUES (".$insert[1].")";
$connection->queryExecute($sql);

// Подготовка строки для использования в sql-запросе (максимальная длина - 50 символов)
$sqlHelper = $connection->getSqlHelper();
$string = $sqlHelper->forSql($string, 50);

// Скалярный запрос (возвращает не набор данных, а конкретное значение)
$count = $connection->queryScalar("SELECT COUNT(ID) FROM table_name");

```