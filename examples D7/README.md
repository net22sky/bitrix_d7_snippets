## Главный модуль ##

### Подключить модуль: ### 

```php
\Bitrix\Main\Loader::includeModule('iblock');

```

### Подключить языковые файлы ###

```php
\Bitrix\Main\Localization\Loc::getMessage('UPDATE_DATE_TEXT');

```

Подключить js, css в шаблоне

```php
\Bitrix\Main\Page\Asset::getInstance()->addCss("/path/to/style.css");  
\Bitrix\Main\Page\Asset::getInstance()->addJs("/path/to/script.js");

```

### Получить настройку модуля: ###
```php

$option = \Bitrix\Main\Config\Option::get('main', 'rating_normalization_type');

```

Запись в лог и время выполнения

use Bitrix\Main\Diag\Debug;
Debug::dumpToFile($arResult);

Debug::startTimeLabel("catalogFilter"); 
#code
Debug::endTimeLabel("catalogFilter"); 
Debug::dumpToFile(Debug::getTimeLabels()); 
Вытащить поля пользователя

$dbUser = \Bitrix\Main\UserTable::getList(array(
	'select' => array('ID', 'NAME', 'PERSONAL_PHOTO', 'PERSONAL_WWW'),
	'filter' => array('ID' => $USER->GetID())
));
if ($arUser = $dbUser->fetch()){
	var_dump($arUser);
}
Пользовательские поля

$dbUserFields = \Bitrix\Main\UserFieldTable::getList(array(
	'filter' => array('ENTITY_ID' => 'IBLOCK_'.$iblockId.'_SECTION')
));
while ($arUserField = $dbUserFields->fetch()) {
	if ($arUserField["USER_TYPE_ID"] == 'enumeration') {
		$dbEnums = CUserFieldEnum::GetList(
			array(),
			array('USER_FIELD_ID' => $arUserField['ID'])
		);
		while ($arEnum = $dbEnums->GetNext()) {
			$arUserField['ENUMS'][$arEnum['XML_ID']] = $arEnum;
		}
	}
	$arResult['USER_FIELDS'][$arUserField["FIELD_NAME"]] = $arUserField;
}
Удобное управление группами пользователя

// Получение групп пользователя 
$dbGroup = \Bitrix\Main\UserGroupTable::getList(array(
	'filter' => array("USER_ID" => 90)
));
$arGroup = $dbGroup->FetchAll();	

// Добавить пользователя в группу
\Bitrix\Main\UserGroupTable::add(array(
	"USER_ID" => $order['USER_ID'],
	"GROUP_ID" => 10,
));
 
// Удалить пользователя из группы  
\Bitrix\Main\UserGroupTable::delete(array(		 
	"USER_ID" => $order['USER_ID'], 
	"GROUP_ID" => 10, 
) );
Кеширование PHP

$cache = Bitrix\Main\Data\Cache::createInstance(); 
if ($cache->initCache($cacheTime, $cacheId, $cacheDir)) { 
	$arResult = $cache->getVars(); 
} elseif ($cache->startDataCache()) { 
	$arResult = array(); 
	if ($isInvalid) { 
		$cache->abortDataCache(); 
	} 
	$cache->endDataCache($arResult); 
} 
