# bitrix_d7_snippets


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

## Сайт URL

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

