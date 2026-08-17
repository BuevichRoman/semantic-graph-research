# Backend Semantic Graph — Research Pass 10
# `query_roles` Consumer Trace v0.1

**Статус:** PARTIALLY ANSWERED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-09 `query_roles` Deep Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Исследовательский вопрос

> В конкретном consumer'е `query_roles` что именно ограничивает: право выполнения операции, набор данных/строк, SQL-конструкцию или другой query scope?

## 2. Найденные consumers
### `archive_17012026_1259/taxi/models/api.php` — `queryString`
```php
17805: 				$s = $sql_insert;
17806: 				$q = query($s);
17807: 
17808: 				if ($q === false) return $this->showError('404', 'error', 'database insert failed');
17809: 			}
17810: 			else
17811: 			{
17812: 				if ($c_product_count > 0)
17813: 				{
17814: 					$u_product_count = $c_product_count + $product_count;
17815: 					if ($u_product_count > 0)
17816: 					{
17817: 						$s = $sql_update;
17818: 						$q = query($s);
17819: 						
17820: 						if ($q === false) return $this->showError('404', 'error', 'update failed');
17821: 					}
17822: 					else
17823: 					{
17824: 						$s = $sql_delete;
17825: 						$q = query($s);
17826: 						
17827: 						if ($q === false) return $this->showError('404', 'error', 'cart delete failed');
17828: 					}
17829: 				}
17830: 				elseif ($c_product_count < 0)
17831: 				{
17832: 					$s = $sql_delete;
17833: 					$q = query($s);
17834: 					
17835: 					if ($q === false) return $this->showError('404', 'error', 'delete failed');
17836: 				}
17837: 			}
17838: 
17839: 			$q = query("COMMIT");
17840: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
17841: 
17842: 			$out = array();
17843: 
17844: 			if (DEFAULT_PROFILE == 'stadium')
17845: 			{
17846: 				if ($product_count > 0)
17847: 				{
17848: 					$s = "UPDATE `cart`
17849: 					LEFT JOIN `cart` as c ON c.`product` = `cart`.`product` AND c.`property` = `cart`.`property` AND c.`id_user` != '" . $_SESSION[UID] . "' AND c.`booking_limit` > now()
17850: 					SET 
17851: 						`cart`.`booking_limit` = IF((SELECT
17852: 								1
17853: 							FROM `ticket`
17854: 							WHERE 
17855: 								`ticket`.`id_trip` = `cart`.`product` AND `ticket`.`id_trip_seat` = `cart`.`property` AND (`ticket`.`id_order` IS NOT NULL OR `ticket`.`status` in (2,3))
17856: 							LIMIT 1
17857: 						) = 1,0,now() + INTERVAL " . $this->constant['ticket_booking_duration'] . " SECOND)				
17858: 					WHERE  
17859: 						`cart`.`id_user` = '" . $_SESSION[UID] . "' AND c.`id_user` IS NULL AND `cart`.`complex_update` = 0
17860: 					";
17861: 
17862: 					$q = query($s);
17863: 					if ($q === false) $out['warning'][] = 'limit update failed';	
17864: 					
17865: 					$s = "DELETE
17866: 						FROM `cart`
17867: 						WHERE 
17868: 							`id_user` = '" . $_SESSION[UID] . "' AND 
17869: 							`booking_limit` < now()
17870: 						";
17871: 
17872: 					$q = query($s);
17873: 					if ($q === false) $out['warning'][] = 'limit delete failed';
17874: 				}
17875: 			}
17876: 
17877: 			return array(
17878: 				'code' 		=>	'200',
17879: 				'status' 	=>	'success',
17880: 				'data' 		=>	$out
17881: 			);
17882: 		}
17883: 
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
```

### `archive_17012026_1259/taxi/cache/data.php` — `unknown`
```php
13766:       'about_fr' => '',
13767:       'about_es' => '',
13768:       'value' => '+41(2342)1234233',
13769:     ),
13770:     'site_contact_email' => 
13771:     array (
13772:       'group' => '1',
13773:       'ru' => 'Контактный емейл',
13774:       'en' => NULL,
13775:       'ar' => NULL,
13776:       'fr' => NULL,
13777:       'es' => NULL,
13778:       'about_ru' => '',
13779:       'about_en' => '',
13780:       'about_ar' => '',
13781:       'about_fr' => '',
13782:       'about_es' => '',
13783:       'value' => 'info@tickets.com',
13784:     ),
13785:     'email_date_format' => 
13786:     array (
13787:       'group' => '1',
13788:       'ru' => 'Формат для вывода даты в емейлах.',
13789:       'en' => 'Format for displaying dates in emails.',
13790:       'ar' => NULL,
13791:       'fr' => NULL,
13792:       'es' => NULL,
13793:       'about_ru' => 'Информация о параметрах: https://www.php.net/manual/ru/datetime.format.php',
13794:       'about_en' => 'Parameter information: https://www.php.net/manual/en/datetime.format.php',
13795:       'about_ar' => '',
13796:       'about_fr' => '',
13797:       'about_es' => '',
13798:       'value' => 'Y-m-d H:i:s',
13799:     ),
13800:     'data_edit_users_roles_rights' => 
13801:     array (
13802:       'group' => '1',
13803:       'ru' => 'Массив таблиц справочника, доступных редактирования юзером с определенной ролью',
13804:       'en' => NULL,
13805:       'ar' => NULL,
13806:       'fr' => NULL,
13807:       'es' => NULL,
13808:       'about_ru' => 'Значение true для роли означает полный доступ, в остальных случаях только доступ к таблицам, чьи веб псевдонимы указаны в ключах массива.',
13809:       'about_en' => '',
13810:       'about_ar' => '',
13811:       'about_fr' => '',
13812:       'about_es' => '',
13813:       'value' => '{"4":true,"1":{},"2":{"stadiums":true,"schedule":true}}',
13814:     ),
13815:     'stripe_seat_title_template' => 
13816:     array (
13817:       'group' => '14',
13818:       'ru' => 'Шаблона названия места в страйпе',
13819:       'en' => NULL,
13820:       'ar' => NULL,
13821:       'fr' => NULL,
13822:       'es' => NULL,
13823:       'about_ru' => '',
13824:       'about_en' => '',
13825:       'about_ar' => '',
13826:       'about_fr' => '',
13827:       'about_es' => '',
13828:       'value' => 'if{$row=="-1"}`Category {$block}``Category {$block}, Seat - {$seat}`',
13829:     ),
13830:     'order_payment' => 
13831:     array (
13832:       'group' => '3',
13833:       'ru' => 'Методы оплаты заказа',
13834:       'en' => NULL,
13835:       'ar' => NULL,
13836:       'fr' => NULL,
13837:       'es' => NULL,
13838:       'about_ru' => 'Формировать ли ссылку для оплаты и каким сервисом. Пустая строка или 0 значит, что заказ создается без оплаты. На выбор доступно 0,1.',
13839:       'about_en' => '',
13840:       'about_ar' => '',
13841:       'about_fr' => '',
13842:       'about_es' => '',
13843:       'value' => '1',
13844:     ),
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
13945: }',
13946:     ),
13947:     'tracker_model2' => 
13948:     array (
13949:       'group' => '3',
13950:       'ru' => 'Сигналы тренера   model2',
13951:       'en' => NULL,
13952:       'ar' => NULL,
13953:       'fr' => NULL,
13954:       'es' => NULL,
13955:       'about_ru' => '',
13956:       'about_en' => '',
13957:       'about_ar' => '',
13958:       'about_fr' => '',
13959:       'about_es' => '',
13960:       'value' => '{
13961:     "type": [
13962:         "e_alarm_speed",
13963:         "e_alarm_out_fence",
13964:         "e_alarm_in_fence",
13965:         "e_alarm_lowpower",
```

### `archive_17012026_1259/taxi/controllers/c_index.php` — `unknown`
```php
3285: 				"u_currency":		"iso4217 код валюты, выбранной пользователем",		data.currencies
3286: 				"u_gps_software":	"идентификатор навигации, выбранной пользователем	data.gps_softwares
3287: 			}
3288: 			Дата в виде "год-месяц-день час:минуты:секунды±часы:минуты</pre>
3289: 		<fieldset class="form"><legend title="Корзина пользователя. Доступна только для авторизованного пользователя. Только для клиента и водителя.">Корзина с забронированными товарами.</legend>
3290: 			<form action="api/v1/cart" method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3291: 				<label class="no_border exclude"><span>включить фильтр</span><input data-name="filter" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3292: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3293: 			</form>
3294: 		</fieldset>
3295: 		<pre>
3296: 		https://ibronevik.ru/taxi/api/v1/cart?prod={t_id}&prop={seat}[&count=1]		GET
3297: 			Управление корзиной			
3298: 				Доступно только для авторизованного пользователя.
3299: 				Параметры запроса:				
3300: 					prod		для default_profile=stadium идентификатор рейса
3301: 					prop		для default_profile=stadium номер места
3302: 					count		для default_profile=stadium 0|1, необязательно
3303: 				Значения параметра 	count:
3304: 					0 			удаление товара из корзины
3305: 					&gt;0			добавление указанного количества или прибавление к существующему
3306: 					&lt;0			уменьшение на указанное количество или удаление
3307: 
3308: 				Билеты с ticket_status=2,3 забронировать нельзя
3309: 				При добавлении товара дата бронирования этого товара назначается всем остальным в корзине.
3310: 				Товары: 
3311: 					дата бронирования которых прошла и они уже забронированы другими пользователями
3312: 					проданы
3313: 					с ticket_status=2,3
3314: 				удаляются.
3315: 				Ответ сервера:
3316: 				{'code':'200','status':'success'}</pre>
3317: 		<fieldset class="form"><legend title="Управление корзиной.">Управляет корзиной</legend>
3318: 			<form action="api/v1/cart"  method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3319: 				<label><span>товар или идентификатор рейса</span><input data-name="prod" name="prod" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3320: 				<label><span>свойство товара или номер места</span><input data-name="prop" name="prop" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3321: 				<label class="exclude"><span>количество товара</span><input data-name="count" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3322: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3323: 			</form>
3324: 		</fieldset>
3325: 		<pre>
3326: 	https://ibronevik.ru/taxi/api/v1/cart/clear								GET
3327: 		Очистка корзина пользователя.		
3328: 			Доступно только для авторизованного пользователя.
3329: 			Параметры запроса:
3330: 				item				JSON.stringify({"prod":["prop",...],...}), POST	
3331: 			Ответ сервера:
3332: 			{'code':'200','status':'success'}</pre>
3333: 		<fieldset class="form"><legend title="Очистка корзины пользователя.">Очищает корзину пользователя.</legend>
3334: 			<form action="api/v1/cart/clear" method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3335: 				<label class="exclude"><span>массив элементов для очистки</span><input data-name="item" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3336: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3337: 			</form>
3338: 		</fieldset>
3339: 		<pre>
3340: 	https://ibronevik.ru/taxi/api/v1/cart/move								GET
3341: 		Передача корзины другому пользователю.		
3342: 			Доступно только для авторизованного пользователя.
3343: 			Параметры запроса:
3344: 				u_id				идентификатор пользлвателя, которому передаются элементы корзины
3345: 				item				JSON.stringify({"prod":["prop",...],...}), POST	
3346: 			Ответ сервера:
3347: 			{'code':'200','status':'success'}</pre>
3348: 		<fieldset class="form"><legend title="Передача корзины пользователя.">Передает корзину пользователя.</legend>
3349: 			<form action="api/v1/cart/move" method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3350: 				<label><span>идентификатор пользлвателя, которому передаются элементы корзины</span><input name="u_id" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3351: 				<label class="exclude"><span>массив элементов для очистки</span><input data-name="item" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3352: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3353: 			</form>
3354: 		</fieldset>
3355: 		<pre>
3356: 		https://ibronevik.ru/taxi/api/v1/query/select/								GET
3357: 		https://ibronevik.ru/taxi/api/v1/query/update/								GET
3358: 		https://ibronevik.ru/taxi/api/v1/query/insert/								GET
3359: 		https://ibronevik.ru/taxi/api/v1/query/delete/								GET
3360: 		https://ibronevik.ru/taxi/api/v1/query/replace/								GET
3361: 		https://ibronevik.ru/taxi/api/v1/query/custom/								GET
3362: 			Запрос на получение данных из базы		
3363: 				Доступно только для авторизованного пользователя.
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
3377: 					{$_SYS[AUTH][u_id]}				идентификатор юзера
3378: 					{$_SYS[AUTH][u_name]}			имя юзера
3379: 					{$_SYS[AUTH][u_family]}			фамилия юзера
3380: 					{$_SYS[AUTH][u_middle]}			отчество юзера
3381: 					{$_SYS[AUTH][u_email]}			емейл юзера, для NULL строка 'NULL'
3382: 					{$_SYS[AUTH][u_phone]}			телефон юзера, для NULL строка 'NULL'
3383: 					{$_SYS[AUTH][u_tg]}				идентификатор телеграма юзера, для NULL строка 'NULL'
3384: 					{$_SYS[AUTH][u_role]}			идентификатор роли юзера в базе
3385: 					{$_SYS[AUTH][u_a_role]}			идентификатор роли юзера для запроса, для NULL строка 'NULL'
3386: 					{$_SYS[AUTH][u_check_state]}	статус верификации юзера, для NULL строка 'NULL'
3387: 					{$_SYS[AUTH][u_ban]}			забанен ли юзер, 0|1
3388: 					{$_SYS[AUTH][u_active]}			статус активности юзера
3389: 					{$_SYS[AUTH][u_birthday]}		дата рождения юзера, для NULL строка 'NULL'
3390: 					{$_SYS[AUTH][u_lang]}			идентификатор языка юзера, для NULL строка 'NULL'
3391: 					{$_SYS[AUTH][lang]}				идентификатор языка для запроса
3392: 					{$_SYS[AUTH][u_currency]}		iso4217 код валюты юзера в верхнем регистре, для NULL строка 'NULL'
3393: 				Для произвольного sql (custom) можно указыать несколько выражений, разделенных точкой с запятой.
3394: 				Ответ сервера:
3395: 				{'code':'200','status':'success','data:{...}}</pre>
3396: 		<fieldset class="form"><legend title="Получение данных.">Выполняет select запрос в базу</legend>
3397: 			<form id="query" data-action="api/v1/query/" action="api/v1/query/"  method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3398: 				<label><span>запрос</span><textarea data-name="sql" name="sql"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3399: 				<label>
3400: 					<span>Комманда</span>				
3401: 					<select id="query_statement">
3402: 						<option value="select" selected>SELECT</option>
3403: 						<option value="update">UPDATE</option>
3404: 						<option value="insert">INSERT</option>
3405: 						<option value="delete">DELETE</option>
3406: 						<option value="replace">REPLACE</option>
3407: 						<option value="custom">CUSTOM</option>
3408: 					</select>
3409: 				<label class="exclude"><span>название sql переменной, значение которой будет возвращено</span><input data-name="var" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3410: 				</label>
3411: 				<label class="exclude"><span>хеш</span><input data-name="hash" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3412: 				</label>
3413: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3414: 			</form>
3415: 		</fieldset>
3416: 		<pre>
3417: 		https://ibronevik.ru/taxi/api/v1/query/template/{template_id}				POST
3418: 			Выполнение шаблонного запроса	
3419: 				Доступно только для авторизованного пользователя.
3420: 				Доступно администратору (непустое only_admin шаблона) или любому пользователю (пустое only_admin шаблона).
3421: 				Параметры запроса:
3422: 					data=JSON.stringify({"метка для подстановки в шаблоне":"значение",...})
3423: 					var			название sql переменной, значение которой будет возвращено, GET или POST
3424: 				Ключ ответа сервера data содержит массив.
3425: 				Если запрос возвращает true, то есть не предполагает вывода строк, то в data:
3426: 					id			автоматически созданное значени инкримента
3427: 					rows		число измененных строк
3428: 				Если var определена и не пустая строка, то в data добавляется  элемент массива с ключами:
3429: 					is_var		1
3430: 					var			значение sql переменной
3431: 				В коде шаблона можно импользовать метки, в которые подставлются данные авторизованного юзера:
3432: 					{$_SYS[AUTH][u_id]}				идентификатор юзера
3433: 					{$_SYS[AUTH][u_name]}			имя юзера
3434: 					{$_SYS[AUTH][u_family]}			фамилия юзера
3435: 					{$_SYS[AUTH][u_middle]}			отчество юзера
3436: 					{$_SYS[AUTH][u_email]}			емейл юзера, для NULL строка 'NULL'
3437: 					{$_SYS[AUTH][u_phone]}			телефон юзера, для NULL строка 'NULL'
3438: 					{$_SYS[AUTH][u_tg]}				идентификатор телеграма юзера, для NULL строка 'NULL'
3439: 					{$_SYS[AUTH][u_role]}			идентификатор роли юзера в базе
3440: 					{$_SYS[AUTH][u_a_role]}			идентификатор роли юзера для запроса, для NULL строка 'NULL'
3441: 					{$_SYS[AUTH][u_check_state]}	статус верификации юзера, для NULL строка 'NULL'
3442: 					{$_SYS[AUTH][u_ban]}			забанен ли юзер, 0|1
3443: 					{$_SYS[AUTH][u_active]}			статус активности юзера
3444: 					{$_SYS[AUTH][u_birthday]}		дата рождения юзера, для NULL строка 'NULL'
3445: 					{$_SYS[AUTH][u_lang]}			идентификатор языка юзера, для NULL строка 'NULL'
3446: 					{$_SYS[AUTH][lang]}				идентификатор языка для запроса
3447: 					{$_SYS[AUTH][u_currency]}		iso4217 код валюты юзера в верхнем регистре, для NULL строка 'NULL'
3448: 				При выполнении шаблона проверяется, содержится ли выражениеЮ с которого он начинается в sql_templates.statement.
3449: 			Ответ сервера:
3450: 			{'code':'200','status':'success','data:{...}}</pre>
3451: 		<fieldset class="form"><legend title="Выполняет шаблонный запрос.">Выполнение шаблона</legend>
3452: 			<form id="query_template" class="complex" action="api/v1/query/template/" data-action="api/v1/query/template/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3453: 				<label class="no_border"><span>идентификатор шаблона (data.sql_temp
```

### `archive_17012026_1259/taxi/controllers/c_index.php` — `get_fields`
```php
9190: 			можно теперь указывать в виде:
9191: 				JSON.stringify({"succeeded":"страница для успешной оплаты","failed":"страница для отменной оплаты"})
9192: 			При указании через GET параметр важно не забыть encodeURIComponent
9193: 
9194: 		Поправлено заполнение шаблонов, когда опции блоков не обновлялись в каждом цикле и возникало перезаполнение памяти
9195: 
9196: 		https://ibronevik.ru/taxi/api/v1/drive										POST
9197: 			Добавлен проброс емейла клиента при формировании ссылки на оплату
9198: 			Более расширена обработка константы stripe_seat_title_template (учет row=-1)
9199: 
9200: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}							POST
9201: 			Для DEFAULT_PROFILE=stadium и set_cancel_state происходит 
9202: 				проверка b_payment_sum
9203: 				проверка b_options.tickets.payment
9204: 				отмена ссылки на оплату
9205: 				в зависимости от результата отмены устанавливается статус
9206: 				
9207: 		Для TikShow крона скрипта отмены просроченных платежей
9208: 			отмена ссылки на оплату
9209: 			в зависимости от результата отмены устанавливается статус
9210: 			
9211: 		https://ibronevik.ru/taxi/api/v1/data/										GET
9212: 			Добавлен параметр
9213: 				json_like
9214: 
9215: 		Для конвертации html в pdf добавлен шрифт Inter
9216: 		</pre>
9217: 		<pre>
9218: 	Обновление (сентябрь 2024)
9219: 		https://ibronevik.ru/taxi/api/v1/drive										POST			
9220: 			Добавлено:
9221: 				u_id
9222: 			Также адмим может создавать заказы без оплаты
9223: 		</pre>
9224: 		<pre>
9225: 	Обновление 2 (сентябрь 2024)
9226: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}							POST
9227: 			Для set_cancel_state даны права админу и добавлен параметр forced
9228: 		</pre>
9229: 		<pre>
9230: 	Обновление 3 (сентябрь 2024)
9231: 		Все юзеры, кроме админа и билетера, не могу изменять свою роль на билетера параметром u_a_role
9232: 
9233: 		https://ibronevik.ru/taxi/api/v1/user							POST
9234: 			Администратор теперь может редактировать:
9235: 				"sc_id"
9236: 			Билетер не может редактировать ничего.
9237: 			Все юзеры, кроме админа и билетера, могут менять на любую роль, кроме админа и билетера.
9238: 			
9239: 		https://ibronevik.ru/taxi/api/v1/user/{u_id1,u_id5}
9240: 		https://ibronevik.ru/taxi/api/v1/user/authorized
9241: 		https://ibronevik.ru/taxi/api/v1/user
9242: 		https://ibronevik.ru/taxi/api/v1/user/authorized/favorite
9243: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/favorite	
9244: 		https://ibronevik.ru/taxi/api/v1/user/authorized/referral
9245: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/referral
9246: 		https://ibronevik.ru/taxi/api/v1/user/authorized/inner
9247: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/inner
9248: 			Добавлено:
9249: 				sc_id
9250: 
9251: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/edit/	POST			
9252: 			Добавлен параметр
9253: 				pass
9254: 
9255: 		https://ibronevik.ru/taxi/api/v1/ticket/check/					GET
9256: 			Добавлен метод проверки билета по коду.
9257: 			
9258: 		https://ibronevik.ru/taxi/api/v1/schedule/ticket/select			GET
9259: 			Поля code_qr_base64 и code выводятся теперь по определенным критериям
9260: 
9261: 		https://ibronevik.ru/taxi/api/v1/data/							POST
9262: 			Для водителя добавлено требование check_state=2
9263: 
9264: 		https://ibronevik.ru/taxi/api/v1/query/select/
9265: 		https://ibronevik.ru/taxi/api/v1/query/update/
9266: 		https://ibronevik.ru/taxi/api/v1/query/insert/
9267: 		https://ibronevik.ru/taxi/api/v1/query/delete/
9268: 		https://ibronevik.ru/taxi/api/v1/query/replace/
9269: 			Разрешение выполняться определенной ролью регулируется теперь константой data.site_constants.query_roles.
9270: 		</pre>
9271: 		<pre>
9272: 	Обновление (октябрь 2024)
9273: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
9274: 			В основной блок добавлены массивы:
9275: 				"aggregators":{...}
9276: 				"field_types":{...}
9277: 				"ticket_props":{...}
9278: 				"booking_props":{...}
9279: 				"promocodes":{...}
9280: 				"users_props":{...}				
9281: 
9282: 		https://ibronevik.ru/taxi/api/v1/register/						POST
9283: 			Добавлено:
9284: 				u_wa
9285: 
9286: 		Шаблоны емейлов:	
9287: 			Регистрация:
9288: 				Добавлена переменная 
9289: 					{$u_wa}
9290: 
9291: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
9292: 			Добавлен type:
9293: 				wa
9294: 				
9295: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}					POST
9296: 		https://ibronevik.ru/taxi/api/v1/user							POST
9297: 			Администратор теперь может редактировать:
9298: 				"u_wa"
9299: 				"u_wa_checked"
9300: 
9301: 		https://ibronevik.ru/taxi/api/v1/user/{u_id1,u_id5}
9302: 		https://ibronevik.ru/taxi/api/v1/user/authorized
9303: 		https://ibronevik.ru/taxi/api/v1/user
9304: 		https://ibronevik.ru/taxi/api/v1/user/authorized/favorite
9305: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/favorite	
9306: 		https://ibronevik.ru/taxi/api/v1/user/authorized/referral
9307: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/referral
9308: 		https://ibronevik.ru/taxi/api/v1/user/authorized/inner
9309: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}/inner
9310: 			Добавлено:
9311: 				u_wa
9312: 				u_wa_checked
9313: 
9314: 		https://ibronevik.ru/taxi/api/v1/remind/						POST
9315: 			Добавлены
9316: 				u_wa
9317: 
9318: 		Администратор теперь может заходить как любой пользователь с помощью POST параметра:
9319: 				u_a_wa		идентификатор ватсапа пользователя
9320: 		Приоритет использования параметров для авторизации:
9321: 				u_a_id
9322: 				u_a_email
9323: 				u_a_phone
9324: 				u_a_tg
9325: 				u_a_wa
9326: 
9327: 		Доработан вывод для параметра au=e, убрано:
9328: 			u_tg
9329: 			u_tg_checked
9330: 			u_upper
9331: 			sc_id
9332: 			u_wa
9333: 			u_wa_checked
9334: 
9335: 		https://ibronevik.ru/taxi/api/v1/drive										POST			
9336: 			Добавлено:
9337: 				b_pc
9338: 
9339: 		https://ibronevik.ru/taxi/api/v1/promocode/check/							GET
9340: 			Добавлен метод проверки промокода
9341: 
9342: 		Добавлен GET или POST параметр:
9343: 			s_token	
9344: 		</pre>
9345: 		<pre>
9346: 	Обновление (декабрь 2024)
9347: 		https://ibronevik.ru/taxi/api/v1/ticket/check/								GET
9348: 			Добавлено:
9349: 				t_id
9350: 				status
9351: 				b_payment_datetime
9352: 				b_state
9353: 				
9354: 		https://ibronevik.ru/taxi/api/v1/data/										GET
9355: 			Добавлено для nocache:
9356: 				top
9357: 				options
9358: 				time_zone
9359: 				price_time_function
9360: 				currency
9361: 				currency_priority
9362: 				fee
9363: 				tariff
9364: 				tariff_priority
9365: 				code_ean_base64
9366: 
9367: 		Добавлена правильная обработка неправильных коротких u_hash и auth_hash
9368: 	
9369: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/edit/	POST			
9370: 			Для параметра pass добавлено логирование действий билетера
9371: 
9372: 		https://ibronevik.ru/taxi/api/v1/ticket/check/log					GET
9373: 			Добавлен метод вывода логов.
9374: 								
9375: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
9376: 			В основной блок добавлены массивы:
9377: 				"map_place_polygons":{...}
9378: 				"favorite_addresses":{...}
9379: 			Ключи для каждого объекта map_place_polygons:
9380: 				upper
9381: 				var
9382: 				coordinates
9383: 				<?php
9384: 					$sql_name = $sql_description = array();			
9385: 					foreach (taxi::$data['langs'] as $lang)
9386: 					{
9387: 						$sql_name[] = $lang['iso'];
9388: 						$sql_description[] = "about_{$lang['iso']}";
9389: 					}
```

### `archive_17012026_1259/taxi/controllers/c_api.php` — `unknown`
```php
409: 								check_auth_user(); $API->id_role = taxi::$id_role;
410: 								$out = $API->getDropboxFileData(trim($_GET['par'][3]),isset($_REQUEST['private'])?trim($_REQUEST['private']):'',isset($_REQUEST['deleted'])?trim($_REQUEST['deleted']):'');
411: 							}
412: 						}
413: 					}
414: 					elseif ($_GET['par'][2] == 'update')
415: 					{
416: 						if (isset($_REQUEST['code']) && !(empty($_POST['hash'])))
417: 						{
418: 							$out = $API->updateDropbox(trim($_REQUEST['code']),trim($_POST['hash']));
419: 						}
420: 					}
421: 				}			
422: 			}
423: 			elseif ($_GET['par'][1] == 'cart')
424: 			{
425: 				check_auth_user(); $API->id_role = taxi::$id_role; 
426: 				if (isset($_REQUEST['s_token'])) $API->session_token = trim($_REQUEST['s_token']); 
427: 				if (empty($_GET['par'][2]))
428: 				{
429: 					if(empty($_REQUEST['prod']))
430: 					{
431: 						$out = $API->selectCart(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
432: 					}
433: 					else
434: 					{
435: 						$out = $API->updateCart(trim($_REQUEST['prod']),isset($_REQUEST['prop'])?trim($_REQUEST['prop']):NULL,isset($_REQUEST['count'])?(int)$_REQUEST['count']:1,isset($_POST['item'])?$_POST['item']:'');
436: 					}
437: 				}
438: 				else
439: 				{
440: 					if ($_GET['par'][2] == 'clear')
441: 					{
442: 						$out = $API->clearCart(isset($_POST['item'])?$_POST['item']:'');
443: 					}
444: 					elseif ($_GET['par'][2] == 'move')
445: 					{
446: 						$out = $API->moveCart(isset($_POST['item'])?$_POST['item']:'',isset($_REQUEST['u_id'])?trim($_REQUEST['u_id']):NULL);
447: 					}
448: 				}
449: 				$out['ticket_booking_duration'] = empty(taxi::$data['site_constants']['ticket_booking_duration'])?3600:abs((int)taxi::$data['site_constants']['ticket_booking_duration']['value']);
450: 			}
451: 			elseif ($_GET['par'][1] == 'cart_block')
452: 			{
453: 				check_auth_user(); $API->id_role = taxi::$id_role;
454: 				if (empty($_GET['par'][2]))
455: 				{
456: 					if(empty($_REQUEST['prod']))
457: 					{
458: 						$out = $API->selectCartBlock(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
459: 					}
460: 					else
461: 					{
462: 						$out = $API->updateCartBlock(trim($_REQUEST['prod']),isset($_REQUEST['prop'])?trim($_REQUEST['prop']):NULL,isset($_REQUEST['count'])?(int)$_REQUEST['count']:1,isset($_REQUEST['notice'])?(empty($_REQUEST['notice'])?0:1):1,taxi::$data,taxi::$data_stt);
463: 					}
464: 				}
465: 				else
466: 				{
467: 					if ($_GET['par'][2] == 'clear')
468: 					{
469: 						$out = $API->clearCartBlock(isset($_POST['item'])?$_POST['item']:'');
470: 					}
471: 					elseif ($_GET['par'][2] == 'status')
472: 					{
473: 						$out = $API->statusCartBlock(isset($_POST['data'])?$_POST['data']:'');
474: 					}
475: 				}
476: 			}
477: 			elseif ($_GET['par'][1] == 'query')
478: 			{
479: 				check_auth_user(); $API->id_role = taxi::$id_role;
480: 				if (!empty($_GET['par'][2]))
481: 				{
482: 					if ($_GET['par'][2] == 'template')
483: 					{
484: 						$out = $API->queryTemplate(isset($_POST['data'])?$_POST['data']:'',empty($_GET['par'][3])?'':trim($_GET['par'][3]),isset($_REQUEST['var'])?trim($_REQUEST['var']):NULL,isset(taxi::$data_private['sql_templates'])?taxi::$data_private['sql_templates']:array());
485: 					}
486: 					else
487: 					{
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
506: 			}
507: 			elseif ($_GET['par'][1] == 'email')
508: 			{
509: 				check_auth_user(); $API->id_role = taxi::$id_role;
510: 				if (!empty($_GET['par'][2]))
511: 				{
512: 					if ($_GET['par'][2] == 'verification')
513: 					{
514: 						if ($_GET['par'][3] == 'start')
515: 						{
516: 							$out = $API->startEmailVerification();
517: 						}
518: 						elseif ($_GET['par'][3] == 'complete')
519: 						{
520: 							$out = $API->completeEmailVerification(isset($_GET['ev_hash'])?trim($_GET['ev_hash']):'');
521: 						}
522: 					}
523: 				}
524: 			}
525: 			elseif ($_GET['par'][1] == 'script')
526: 			{
527: 				check_auth_user(); $API->id_role = taxi::$id_role;
528: 				if (!empty($_GET['par'][2]))
529: 				{
530: 					if ($_GET['par'][2] == 'template')
531: 					{
532: 						$out = $API->includeTemplate(empty($_GET['par'][3])?'':trim($_GET['par'][3]),empty($_REQUEST['is_var'])?false:true,isset(taxi::$data_private['script_templates'])?taxi::$data_private['script_templates']:array());
533: 					}
534: 				}
535: 			}
536: 			elseif ($_GET['par'][1] == 'translate')
537: 			{
538: 				check_auth_user(); $API->id_role = taxi::$id_role;
539: 				$out = $API->translate(isset($_POST['data'])?$_POST['data']:'',isset($_POST['from'])?$_POST['from']:'',isset($_POST['to'])?$_POST['to']:'');
540: 			}
541: 			elseif ($_GET['par'][1] == 'outer_script')
542: 			{
543: 				check_auth_user(); $API->id_role = taxi::$id_role;
544: 				if (!empty($_GET['par'][2]))
545: 				{
546: 					if ($_GET['par'][2] == 'template')
547: 					{
548: 						$out = $API->requestTemplate(empty($_GET['par'][3])?'':trim($_GET['par'][3]),empty($_REQUEST['is_var'])?false:true,isset(taxi::$data_private['outer_script_templates'])?taxi::$data_private['outer_script_templates']:array());
549: 					}
550: 				}
551: 			}
552: 			elseif ($_GET['par'][1] == 'contact')
553: 			{
554: 				check_auth_user(); $API->id_role = taxi::$id_role;
555: 				if (!empty($_GET['par'][2]))
556: 				{
557: 					if ($_GET['par'][2] == 'get')
558: 					{
559: 						$out = $API->selectContact(isset($_POST['co_id'])?trim($_POST['co_id']):NULL,isset($_POST['u_id'])?trim($_POST['u_id']):NULL,isset($_POST['o_type'])?trim($_POST['o_type']):NULL,isset($_POST['co_class'])?trim($_POST['co_class']):NULL,isset(taxi::$data_private['owner_types'])?taxi::$data_private['owner_types']:array(),isset(taxi::$data['langs'])?taxi::$data['langs']:array());
560: 					}
561: 					elseif ($_GET['par'][2] == 'create')
562: 					{
563: 						$out = $API->createContact(isset($_POST['data'])?$_POST['data']:'',isset(taxi::$data['langs'])?taxi::$data['langs']:array());
564: 					}
565: 					elseif ($_GET['par'][2] == 'edit')
566: 					{
567: 						$out = $API->editContact(isset($_POST['data'])?$_POST['data']:'',isset($_REQUEST['co_id'])?trim($_REQUEST['co_id']):NULL,isset(taxi::$data['langs'])?taxi::$data['langs']:array());
568: 					}
569: 					elseif ($_GET['par'][2] == 'message')
570: 					{
571: 						if (!empty($_GET['par'][3]))
572: 						{
573: 							if ($_GET['par'][3] == 'get')
574: 							{
575: 								$out = $API->selectMessage(isset($_POST['m_id'])?trim($_POST['m_id']):NULL,isset($_POST['from_to'])?$_POST['from_to']:'',isset(taxi::$data_private['owner_types'])?taxi::$data_private['owner_types']:array());
576: 							}
577: 							elseif ($_GET['par'][3] == 'send')
578: 							{
579: 								$out = $API->sendMessage(isset($_POST['title'])?trim($_POST['title']):'',isset($_POST['text'])?trim($_POST['text']):'',isset($_POST['m_id'])?trim($_POST['m_id']):NULL,isset($_POST['m_type'])?trim($_POST['m_type']):NULL,isset($_POST['from_to'])?$_POST['from_to']:'',isset($_POST['co_ids'])?$_POST['co_ids']:'',isset($_POST['co_id'])?$_POST['co_id']:NULL,isset($_POST['code'])?$_POST['code']:NULL,isset(taxi::$data_private['owner_types'])?taxi::$data_private['owner_types']:array(),isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
580: 							}
581: 							elseif ($_GET['par'][3] == 'read')
582: 							{
583: 								$out = $API->readMessage(isset($_REQUEST['m_id'])?trim($_REQUEST['m_id']):NULL,isset(taxi::$data_private['owner_types'])?taxi::$data_private['owner_types']:array());
584: 							}
585: 						}
586: 					}
587: 				}
588: 			}
589: 			elseif ($_GET['par'][1] == 'task')
590: 			{
591: 				if (!empty($_GET['par'][2]))
592: 				{
593: 					check_auth_user(); $API->id_role = taxi::$id_role;
594: 					if ($_GET['par'][2] == 'create')
595: 					{
596: 						$out = $API->createTask(isset($_POST['data'])?$_POST['data']:'');
597: 					}
598: 					elseif ($_GET['par'][2] == 'control')
599: 					{
600: 						$out = $API->controlTask(isset($_REQUEST['tl_id'])?trim($_REQUEST['tl_id']):NULL,isset($_REQUEST['action'])?trim($_REQUEST['action']):NULL);
601: 					}
602: 					elseif ($_GET['par'][2] == 'select')
603: 					{
604: 						$out = $API->selectTask(isset($_REQUEST['tl_id'])?trim($_REQUEST['tl_id']):NULL);
605: 					}
606: 					elseif ($_GET['par'][2] == 'log')
607: 					{
608: 						if (!empty($_GET['par'][3]))
```

## 3. Что непосредственно подтверждено

В consumer chain значение `query_roles` передаётся в query-related processing. Это подтверждает dependency:

```text
query_roles
    ↓
query processing
```

Но для вывода `query_roles = authorization` необходимо увидеть момент, где значение преобразуется в конкретное permission/access decision. Если оно лишь формирует SQL WHERE/filters/data scope, это другой semantic relation.

## 4. Нормативное различение

### AUTHORIZATION
```text
Role / identity
    ↓
permission decision
    ↓
ALLOW / REJECT operation
```

### QUERY_SCOPE
```text
Role / context
    ↓
query constraint
    ↓
subset of data / rows / objects
```

Оба механизма могут существовать одновременно.

## 5. Текущий результат

На уровне доступного source corpus этот pass подтверждает `query_roles → query processing`, но не даёт достаточного основания переименовать `query_roles` в `AuthorizationPolicy`.

Если конкретный consumer показывает только query filter, статус:

```text
CONFIRMED: QUERY_PROCESSING_INPUT
UNKNOWN: AUTHORIZATION_SEMANTICS
```

## 6. Связь с Role Permission Matrix

Прямые `$this->id_role` checks остаются отдельным Evidence authorization.

Поэтому текущая модель:

```text
Authentication
      ↓
Role Resolution
      ↓
Runtime API Role
      ├── operation-local permission checks
      │       ↓
      │    ALLOW / REJECT
      │
      └── query_roles
              ↓
          query processing
              ↓
          QUERY SCOPE
```

## 7. Что не фиксируем

Не фиксируем:

```text
query_roles = Authorization Policy
query_roles = Permission Matrix
Role ID = business role name
query_roles absence = unrestricted query
```

## 8. Gap Report

```text
G-10-01
Exact semantic effect of query_roles on query result
STATUS: OPEN

G-10-02
Whether any query_roles consumer performs explicit permission rejection
STATUS: OPEN

G-10-03
Role ID → business role mapping
STATUS: OPEN

G-10-04
Complete Role × Operation matrix
STATUS: OPEN

## 9. MCR

`MCR = NO CHANGE`.

## 10. Следующий шаг

Не расширять поиск по `query_roles` бесконечно. Следующий шаг — выбрать один конкретный consumer с максимальным semantic yield и проследить его до SQL/result boundary:

```text
query_roles
   ↓
consumer
   ↓
SQL condition / filter
   ↓
returned data
```

Если на этой границе нет ALLOW/REJECT, фиксируем `QUERY_SCOPE`, а не authorization.