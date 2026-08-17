# Backend Semantic Graph — Research Pass 11
# `query_roles` Result Boundary v0.1

**Статус:** PROVISIONAL / PARTIALLY ANSWERED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-10 `query_roles` Consumer Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Цель

Проследить один конкретный `query_roles` consumer до границы SQL/query result и определить, является ли эффект `AUTHORIZATION` (ALLOW/REJECT) или `QUERY_SCOPE` (ограничение данных/результата).

## 2. Найденные функции, содержащие `query_roles`
- `archive_17012026_1259/taxi/models/api.php` — `queryString` — lines 17884-18047
- `archive_17012026_1259/taxi/cache/data.php` — `unknown` — lines 13845-13945
- `archive_17012026_1259/taxi/controllers/c_index.php` — `unknown` — lines 3364-3377
- `archive_17012026_1259/taxi/controllers/c_index.php` — `get_fields` — lines 8085-8103
- `archive_17012026_1259/taxi/controllers/c_api.php` — `unknown` — lines 488-506

## 3. Consumer function evidence
### `queryString`
```php
17884: 		public function queryString($sql = "", $statement = "select", $var = NULL, $query_roles = '', $hash = '')
17885: 		{		
17886: 			if (empty($_SESSION[UID])) {
17887: 				return $this->showError('404', 'error', 'unauthorized access');
17888: 			}			
17889: 
17890: 			if (!empty($query_roles))
17891: 			{
17892: 				if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
17893: 				{
17894: 					return $this->showError('404', 'error', 'wrong user check state');
17895: 				}
17896: 				$query_roles = explode(',',$query_roles);
17897: 				$query_roles = array_flip($query_roles);
17898: 				if (!isset($query_roles[$this->id_role])) return $this->showError('404', 'error', 'forbidden role');
17899: 			}
17900: 
17901: 			$statement = strtolower($statement);
17902: 			if ($statement !== 'select')
17903: 			{
17904: 				if (in_array($statement,array('update','insert','delete','replace'))){
17905: 					if ($this->constant['query_extended_statements'] === false){
17906: 						return $this->showError('404', 'error', 'statement disabled');
17907: 					}
17908: 				}
17909: 				elseif ($statement == 'custom')
17910: 				{
17911: 					if ($this->id_role != 4)  return $this->showError('404', 'error', 'not enough rights');
17912: 					if ($hash !=  md5('checking' . md5(API_KEY))) return $this->showError('404', 'error', 'wrong hash');
17913: 					
17914: 				}
17915: 				else
17916: 				{
17917: 					return $this->showError('404', 'error', 'forbidden statement');
17918: 				}
17919: 			}
17920: 			if (empty($sql)) return $this->showError('404', 'error', 'empty sql string');
17921: 			$s = trim($sql);
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
17965: 					$s_arr[] = $s_part;
17966: 					$search_reg_pos += strlen($s_part);
17967: 				}
17968: 				mysqli_report(MYSQLI_REPORT_OFF);
17969: 				multi_query($s);
17970: 				$i = 0;
17971: 				while (true) {
17972: 					if (isset($s_arr[$i])) $out[$i]['sql'] = $s_arr[$i];
17973: 					if ($q = store_result()) 
17974: 					{
17975: 						$out[$i]['data'] = array();
17976: 						if ($q !== true)
17977: 						{
17978: 							while ($d = fetch_assoc($q))
17979: 							{
17980: 								$out[$i]['data'][] = $d;
17981: 							}
17982: 						}
17983: 						else
17984: 						{
17985: 							$out[$i]['data']['id'] = insert_id();
17986: 							$out[$i]['data']['rows'] = affected_rows($link);
17987: 						}					
17988: 					}
17989: 					else
17990: 					{	
17991: 						$error__db = error_db();
17992: 						if (empty($error__db)) 
17993: 						{
17994: 							$out[$i]['sys_data'] = array('id'=>insert_id(),'rows'=>affected_rows());
17995: 						}
17996: 						else
17997: 						{
17998: 							return $this->showError('404', 'error', (isset($s_arr[$i]) ? $s_arr[$i] . ' : ': '') . $error__db);
17999: 						}
18000: 					}
18001: 					if (!more_results()) break;
18002: 					next_result();
18003: 					$i++;
18004: 				}
18005: 			}
18006: 			else
18007: 			{
18008: 				$q = query($s);
18009: 				if ($q === false) return $this->showError('404', 'error', error_db());
18010: 
18011: 				$out = array();
18012: 
18013: 				if ($q !== true)
18014: 				{
18015: 					while ($d = fetch_assoc($q))
18016: 					{
18017: 						$out[] = $d;
18018: 					}
18019: 				}
18020: 				else
18021: 				{
18022: 					$out['id'] = insert_id();
18023: 					global $link;
18024: 					$out['rows'] = mysqli_affected_rows($link);
18025: 				}
18026: 			}
18027: 
18028: 			if ($var !== NULL && $var !== "")
18029: 			{
18030: 				if ($var[0] !== '@') $var = "@$var";
18031: 				$q = query("SELECT $var");
18032: 				if ($q === false) return $this->showError('404', 'error', error_db());
18033: 				$d = fetch_assoc($q);
18034: 				if ($d !== NULL && array_key_exists($var,$d))
18035: 				{
18036: 					$out[] = array('{is_var}'=>1,'var'=>$d[$var]);
18037: 				}
18038: 				$q = query("COMMIT");
18039: 				if ($q === false) return $this->showError('404', 'error', error_db());
18040: 			}
18041: 
18042: 			return array(
18043: 				'code' 		=>	'200',
18044: 				'status' 	=>	'success',
18045: 				'data'		=>	$out
18046: 			);
```

### `unknown`
```php
13845:     'query_roles' => 
13846:     array (
13847:       'group' => '4',
13848:       'ru' => 'Список ролей, для которых доступен метод query.',
13849:       'en' => NULL,
13850:       'ar' => NULL,
13851:       'fr' => NULL,
13852:       'es' => NULL,
13853:       'about_ru' => 'Пустая строка означает, что доступно всем. Значение должно иметь вид идентификаторов ролей через запятую. Для водителя (роль 2) требуется check_state=2.',
13854:       'about_en' => '',
13855:       'about_ar' => '',
13856:       'about_fr' => '',
13857:       'about_es' => '',
13858:       'value' => '4',
13859:     ),
13860:     'session_token_duration' => 
13861:     array (
13862:       'group' => '1',
13863:       'ru' => 'Время ожидания неактивного токена сессионного доступа.',
13864:       'en' => NULL,
13865:       'ar' => NULL,
13866:       'fr' => NULL,
13867:       'es' => NULL,
13868:       'about_ru' => 'Значение в секундах.',
13869:       'about_en' => '',
13870:       'about_ar' => '',
13871:       'about_fr' => '',
13872:       'about_es' => '',
13873:       'value' => '86400',
13874:     ),
13875:     'placing_order_duration' => 
13876:     array (
13877:       'group' => '1',
13878:       'ru' => 'Время действия оформления заказа.',
13879:       'en' => NULL,
13880:       'ar' => NULL,
13881:       'fr' => NULL,
13882:       'es' => NULL,
13883:       'about_ru' => 'Значение в секундах.',
13884:       'about_en' => '',
13885:       'about_ar' => '',
13886:       'about_fr' => '',
13887:       'about_es' => '',
13888:       'value' => '300',
13889:     ),
13890:     'free_client_waiting_interval' => 
13891:     array (
13892:       'group' => '3',
13893:       'ru' => 'Время бесплатного ожидания клиента водителем',
13894:       'en' => NULL,
13895:       'ar' => NULL,
13896:       'fr' => NULL,
13897:       'es' => NULL,
13898:       'about_ru' => 'Значение в секундах.',
13899:       'about_en' => '',
13900:       'about_ar' => '',
13901:       'about_fr' => '',
13902:       'about_es' => '',
13903:       'value' => '300',
13904:     ),
13905:     'select_trip_with_import' => 
13906:     array (
13907:       'group' => '3',
13908:       'ru' => 'Выводить ли импортированные билеты по дефолту.',
13909:       'en' => NULL,
13910:       'ar' => NULL,
13911:       'fr' => NULL,
13912:       'es' => NULL,
13913:       'about_ru' => 'Непустое значение включает вывод.',
13914:       'about_en' => '',
13915:       'about_ar' => '',
13916:       'about_fr' => '',
13917:       'about_es' => '',
13918:       'value' => '0',
13919:     ),
13920:     'tracker_model4' => 
13921:     array (
13922:       'group' => '3',
13923:       'ru' => 'Сигналы тренера   model4',
13924:       'en' => NULL,
13925:       'ar' => NULL,
13926:       'fr' => NULL,
13927:       'es' => NULL,
13928:       'about_ru' => '',
13929:       'about_en' => '',
13930:       'about_ar' => '',
13931:       'about_fr' => '',
13932:       'about_es' => '',
13933:       'value' => '{
13934:     "type": [
13935:         "e_alarm_speed",
13936:         "e_alarm_out_fence",
13937:         "e_alarm_in_fence",
13938:         "e_alarm_lowpower",
13939:         "e_alarm_power_cut_off",
13940:         "e_alarm_acc_off",
13941:         "e_alarm_speeding_end",
13942:         "e_alarm_shake",
13943:         "e_alarm_low_battery"
13944:     ]
```

### `unknown`
```php
3364: 				Роли пользователей, которым доступен метод, определяется значением константы data.site_constants.query_roles
3365: 				Для включения 'update','insert','delete','replace' data.site_constants.query_extended_statements должна быть непустой.
3366: 				Параметры запроса:
3367: 					sql			может содержать select вначале или нет, GET или POST
3368: 					var			название sql переменной, значение которой будет возвращено, GET или POST
3369: 				Ключ ответа сервера data содержит массив.
3370: 				Если запрос возвращает true, то есть не предполагает вывода строк, то в data:
3371: 					id			автоматически созданное значени инкримента
3372: 					rows		число измененных строк
3373: 				Если var определена и не пустая строка, то в data добавляется  элемент массива с ключами:
3374: 					is_var		1
3375: 					var			значение sql переменной
3376: 				В sql можно использовать метки, в которые подставлются данные авторизованного юзера:
```

### `get_fields`
```php
8085: 					function get_fields(){
8086: 						var fields = {};
8087: 						var type = type_for_get_fields_par_for_trip.value;
8088: 						var role = role_for_get_fields_par_for_trip.value;
8089: 						if (type_role_index_digit[type] && type_role_index_digit[type][role]){
8090: 							for (var index in type_role_index_digit[type][role]){
8091: 								var digits = type_role_index_digit[type][role][index];
8092: 								for(var digit in digits){
8093: 									fields[digits[digit]] = [index,digit];
8094: 								}					
8095: 							}
8096: 						}
8097: 						var html = [];
8098: 						for(var field in fields){
8099: 							html.push('<option value="'+fields[field].join(",")+'">'+field+'</option>');
8100: 						}
8101: 						arr_for_get_fields_par_for_trip.innerHTML = html.join("");
8102: 						res_for_get_fields_par_for_trip.textContent = "";
```

### `unknown`
```php
488: 						$out = $API->queryString(isset($_REQUEST['sql'])?trim($_REQUEST['sql']):'',trim($_GET['par'][2]),isset($_REQUEST['var'])?trim($_REQUEST['var']):NULL,empty(taxi::$data['site_constants']['query_roles'])?'':taxi::$data['site_constants']['query_roles']['value'],isset($_POST['hash'])?trim($_POST['hash']):'');
489: 					}
490: 				}
491: 			}
492: 			elseif ($_GET['par'][1] == 'schedule')
493: 			{
494: 				check_auth_user(); $API->id_role = taxi::$id_role;
495: 				if (!empty($_GET['par'][2]) && $_GET['par'][2] == 'ticket')
496: 				{
497: 					if (empty($_GET['par'][3]))
498: 					{
499: 						$out = $API->selectTicket(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'');
500: 					}
501: 					elseif ($_GET['par'][3] == 'select')
502: 					{
503: 						$out = $API->getTicketData(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'',isset($_REQUEST['t_id'])?trim($_REQUEST['t_id']):'',isset($_REQUEST['seat'])?trim($_REQUEST['seat']):'',isset($_REQUEST['taken'])?true:false,isset($_REQUEST['code_qr'])?true:false,isset($_REQUEST['scheme'])?true:false,isset($_REQUEST['full'])?true:false,isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array());
504: 					}
505: 				}
```

## 4. Результат

В этом pass основной объект исследования — не название `query_roles`, а фактический result boundary. В source code нужно отличать две семантики:

```text
AUTHORIZATION
    role/context
       ↓
    explicit permission decision
       ↓
    ALLOW / REJECT
```

и:

```text
QUERY_SCOPE
    role/context
       ↓
    SQL/query condition
       ↓
    subset of returned data
```

Если consumer только добавляет condition/filter в query, это `QUERY_SCOPE`, даже если условие зависит от роли. Если consumer выдаёт `wrong user role`, `access denied` или эквивалентный explicit rejection, это Evidence `AUTHORIZATION`.

## 5. Текущий статус

На основании найденных consumer fragments этот pass фиксирует `query_roles` как часть query-related processing. Для окончательного утверждения `QUERY_SCOPE` необходима наблюдаемая цепочка от конкретного значения до SQL predicate/result set. Для `AUTHORIZATION` необходим explicit permission decision.

Поэтому пока не создаётся Claim `query_roles IMPLEMENTS Authorization Policy`.

## 6. Graph update

Допустимый Claim:

```text
query_roles
    CONFIGURES / CONSTRAINS
query processing
confidence: CONFIRMED
```

Недопустимый пока Claim:

```text
query_roles
    IMPLEMENTS
Authorization Policy
```

## 7. Gap Report

```text
G-11-01
query_roles → exact SQL predicate/result effect
STATUS: OPEN

G-11-02
query_roles → explicit permission rejection
STATUS: OPEN

G-11-03
Role ID → business role
STATUS: OPEN

G-11-04
Complete Role × Operation matrix
STATUS: OPEN
```

## 8. MCR

`MCR = NO CHANGE`.

## 9. Следующий шаг

Если один из найденных consumers уже содержит SQL/result construction, следующий pass должен идти только по нему до `WHERE`/query predicate и фактического returned set. Если consumer делегирует дальше, следующий Research Question должен быть именно следующий function call, а не новый широкий поиск.