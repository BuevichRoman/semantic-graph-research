# Backend Semantic Graph — Research Pass 14
# Role ID Semantic Mapping v0.1

**Статус:** PARTIALLY ANSWERED / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-13 `query_roles` Authorization Boundary v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Исследовательский вопрос

> Что означают числовые `id_role` в production Core Backend, прежде всего `2`, `4`, `5`, `6`?

Цель — получить mapping только там, где business meaning подтверждается source/config/data evidence.

## 2. Результаты поиска

Найдено релевантных source contexts: **237**.

Ниже приведены контексты, которые могут участвовать в role-ID mapping. Само соседство числа и слова не считается доказательством.

### `archive_17012026_1259/taxi/index.php:19`
```text
17: 			show_error('unauthorized access');
18: 		}
19: 		elseif ($_SESSION['id_role'] != 4)
20: 		{
21: 			show_error('not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:23`
```text
21: 				if (empty($_SESSION[UID])) 
22: 				{
23: 					if ($role != 1 && $role != 2 && $role != 5)
24: 					{
25: 						return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:30`
```text
28: 				else
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
32: 					$sql_user = "'" . $_SESSION[UID] . "'";
```

### `archive_17012026_1259/taxi/models/api.php:31`
```text
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
32: 					$sql_user = "'" . $_SESSION[UID] . "'";
33: 				}
```

### `archive_17012026_1259/taxi/models/api.php:37`
```text
35: 			else
36: 			{
37: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
38: 				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
39: 				$sql_user = "'" . $_SESSION[UID] . "'";
```

### `archive_17012026_1259/taxi/models/api.php:38`
```text
36: 			{
37: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
38: 				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
39: 				$sql_user = "'" . $_SESSION[UID] . "'";
40: 			}
```

### `archive_17012026_1259/taxi/models/api.php:661`
```text
659: 			if (empty($id_user))
660: 			{	
661: 				if ($this->id_role != 4)
662: 				{
663: 					$prop_visibility = 4;
```

### `archive_17012026_1259/taxi/models/api.php:685`
```text
683: 				{
684: 					$prop_visibility = 8;
685: 					if ($this->id_role != 4) 
686: 					{
687: 						$sql_add = "";
```

### `archive_17012026_1259/taxi/models/api.php:1056`
```text
1054: 			else
1055: 			{
1056: 				if ($this->id_role != 4) 
1057: 				{
1058: 					return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:1379`
```text
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
```

### `archive_17012026_1259/taxi/models/api.php:1468`
```text
1466: 				if ($data['u_role'] !== $id_role)
1467: 				{
1468: 					if (empty($data['u_role'])) return $this->showError('404', 'error', 'empty user role');
1469: 					if ($_SESSION['id_role'] == 4)
1470: 					{
```

### `archive_17012026_1259/taxi/models/api.php:1469`
```text
1467: 				{
1468: 					if (empty($data['u_role'])) return $this->showError('404', 'error', 'empty user role');
1469: 					if ($_SESSION['id_role'] == 4)
1470: 					{
1471: 						if ($_SESSION[UID] == $id_user)
```

### `archive_17012026_1259/taxi/models/api.php:1485`
```text
1483: 					else
1484: 					{
1485: 						if ($data['u_role'] == 4 || $data['u_role'] == 6 || empty($roles[$data['u_role']]))
1486: 						{
1487: 							return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:1990`
```text
1988: 				if (!empty($d['u_id'])){
1989: 					$d['u_id'] = explode(',',$d['u_id']);
1990: 					if ($this->id_role != 4 && !in_array($_SESSION[UID],$d['u_id']))
1991: 					{
1992: 						unset($d['details']);
```

### `archive_17012026_1259/taxi/models/api.php:1997`
```text
1995: 				else 
1996: 				{
1997: 					if ($this->id_role != 4) unset($d['details']);
1998: 				}
1999: 				if (!empty($d['details'])) {$d['details'] = json_decode($d['details'],true);}
```

### `archive_17012026_1259/taxi/models/api.php:2145`
```text
2143: 				if (empty($id_user) || $_SESSION[UID] == $id_user) 
2144: 				{
2145: 					if ($this->id_role == 2)
2146: 					{
2147: 						if (empty($_SESSION['id_verification_status']) 
```

### `archive_17012026_1259/taxi/models/api.php:2169`
```text
2167: 				else
2168: 				{
2169: 					if ($this->id_role != 4) 
2170: 					{
2171: 						return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:2188`
```text
2186: 
2187: 					if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');
2188: 					if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong role of user');
2189: 					
2190: 					$car = $this->createCar($filtered_data,$id_user);
```

### `archive_17012026_1259/taxi/models/api.php:2200`
```text
2198: 			else
2199: 			{			
2200: 				if (!empty($id_user) && $this->id_role != 4) 
2201: 				{
2202: 					return $this->showError('404', 'error', 'not enough rights for assign');
```

### `archive_17012026_1259/taxi/models/api.php:2225`
```text
2223: 				if (empty($id_user)) 
2224: 				{
2225: 					if ($this->id_role == 4 || (in_array($_SESSION[UID], explode(',',$d['u_id'])) 
2226: 						&& (empty($_SESSION['id_verification_status']) || $_SESSION['id_verification_status'] == 1)))
2227: 					{
```

### `archive_17012026_1259/taxi/models/api.php:2296`
```text
2294: 							FROM `users`
2295: 							WHERE 
2296: 								`id_user` in (" . $id_user . ") AND `id_role` = 2
2297: 							";
2298: 
```

### `archive_17012026_1259/taxi/models/api.php:2849`
```text
2847: 				return $this->showError('404', 'error', 'unauthorized access');
2848: 			}
2849: 			if ($this->id_role != 1 && $this->id_role != 5 && $this->id_role != 4)
2850: 			{
2851: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:3471`
```text
3469: 			if (array_key_exists('u_id',$data))
3470: 			{
3471: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
3472: 
3473: 				$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:3990`
```text
3988: 			elseif ($filtered_data['id_payment_method'] == 4)
3989: 			{
3990: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'forbidden payment way');
3991: 			}
3992: 
```

### `archive_17012026_1259/taxi/models/api.php:4481`
```text
4479: 			}
4480: 
4481: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
4482: 			{
4483: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:4545`
```text
4543: 			}
4544: 
4545: 			if ($this->id_role == 1 || $this->id_role == 5)
4546: 			{
4547: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:4581`
```text
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
```

### `archive_17012026_1259/taxi/models/api.php:4775`
```text
4773: 						if (!empty($d['drivers'][$key]['c_options'])) $d['drivers'][$key]['c_options'] = json_decode($d['drivers'][$key]['c_options'],true);
4774: 
4775: 						if ($this->id_role == 1 || $this->id_role == 5)
4776: 						{
4777: 							if ((int)$d['drivers'][$key]['c_state'] > 1)
```

### `archive_17012026_1259/taxi/models/api.php:4818`
```text
4816: 				}
4817: 
4818: 				if ($this->id_role == 1 || $this->id_role == 5)
4819: 				{
4820: 					add_user_for_order($users_client[$d['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:4852`
```text
4850: 
4851: 				$d['b_options'] = json_decode($d['b_options'],true);
4852: 				if (($this->id_role != 1 && $this->id_role != 5) && is_array($d['b_options']))
4853: 				{
4854: 					unset($d['b_options'][':u_id_alias']);
```

### `archive_17012026_1259/taxi/models/api.php:4948`
```text
4946: 				return $this->showError('404', 'error', 'unauthorized access');
4947: 			}
4948: 			if ($this->id_role != 2 && $this->id_role != 4)
4949: 			{
4950: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:5003`
```text
5001: 			}
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
```

### `archive_17012026_1259/taxi/models/api.php:5020`
```text
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
```

### `archive_17012026_1259/taxi/models/api.php:5067`
```text
5065: 			$sql_limit = "LIMIT " . $limit_offset . ", " . $limit_row_count;
5066: 			$union = false;
5067: 			if (isset($filter) && $this->id_role == 2)
5068: 			{
5069: 				$s = "SELECT 	
```

### `archive_17012026_1259/taxi/models/api.php:5356`
```text
5354: 				}
5355: 			}
5356: 			elseif (isset($filter) && $this->id_role == 2 && $check_license === true)
5357: 			{
5358: 				$i = 0;
```

### `archive_17012026_1259/taxi/models/api.php:5512`
```text
5510: 						}
5511: 
5512: 						if ($this->id_role == 4)
5513: 						{
5514: 							add_user_for_order($users_driver[$d['drivers'][$key]['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:5524`
```text
5522: 				}
5523: 
5524: 				if ($this->id_role == 4)
5525: 				{
5526: 					add_user_for_order($users_client[$d['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:5555`
```text
5553: 
5554: 				$d['b_options'] = json_decode($d['b_options'],true);
5555: 				if ($this->id_role == 2 && is_array($d['b_options']))
5556: 				{
5557: 					unset($d['b_options'][':u_id_alias']);
```

### `archive_17012026_1259/taxi/models/api.php:5608`
```text
5606: 				return $this->showError('404', 'error', 'unauthorized access');
5607: 			}
5608: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
5609: 			{
5610: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:5672`
```text
5670: 			}
5671: 		
5672: 			if ($this->id_role == 1 || $this->id_role == 5)
5673: 			{
5674: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:5854`
```text
5852: 					LEFT JOIN `order_driver` USING (`id_order`)			
5853: 					WHERE
5854: 						(`order`.`id_order_status` in (3,4) OR od.`id_order_driver_status` = '2') AND 
5855: 						od.`id_user` = '" . $_SESSION[UID] . "' 
5856: 					GROUP BY `order`.`id_order`
```

### `archive_17012026_1259/taxi/models/api.php:5911`
```text
5909: 					{
5910: 						$d['drivers'][$key] = array();
5911: 						if ($this->id_role == 1 || $this->id_role == 5)
5912: 						{
5913: 
```

### `archive_17012026_1259/taxi/models/api.php:5986`
```text
5984: 							$d['drivers'][$key]['c_cancel_reason'] = NULL;
5985: 						}
5986: 						if ($d['drivers'][$key]['c_rating'] === chr(2))
5987: 						{
5988: 							$d['drivers'][$key]['c_rating'] = NULL;
```

### `archive_17012026_1259/taxi/models/api.php:5990`
```text
5988: 							$d['drivers'][$key]['c_rating'] = NULL;
5989: 						}
5990: 						if ($d['drivers'][$key]['c_tips'] === chr(2))
5991: 						{
5992: 							$d['drivers'][$key]['c_tips'] = NULL;
```

### `archive_17012026_1259/taxi/models/api.php:6011`
```text
6009: 						if (!empty($d['drivers'][$key]['c_options'])) $d['drivers'][$key]['c_options'] = json_decode($d['drivers'][$key]['c_options'],true);
6010: 
6011: 						if ($this->id_role == 1 || $this->id_role == 5)
6012: 						{
6013: 							if ((int)$d['drivers'][$key]['c_state'] > 1)
```

### `archive_17012026_1259/taxi/models/api.php:6053`
```text
6051: 					}
6052: 				}
6053: 				if ($this->id_role == 1 || $this->id_role == 5)
6054: 				{
6055: 					add_user_for_order($users_client[$d['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:6084`
```text
6082: 
6083: 				$d['b_options'] = json_decode($d['b_options'],true);
6084: 				if (($this->id_role != 1 && $this->id_role != 5) && is_array($d['b_options']))
6085: 				{
6086: 					unset($d['b_options'][':u_id_alias']);
```

### `archive_17012026_1259/taxi/models/api.php:6094`
```text
6092: 				add_time_zone($d['b_start_datetime']);
6093: 				add_time_zone($d['b_created']);
6094: 				if ($this->id_role == 1 || $this->id_role == 5)
6095: 				{
6096: 					add_time_zone($d['b_confirmation_limit']);
```

### `archive_17012026_1259/taxi/models/api.php:6186`
```text
6184: 			}
6185: 
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:6191`
```text
6189: 			}
6190: 			
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
6193: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:6590`
```text
6588: 						return $this->showError('404', 'error', 'busy performers vacancy');
6589: 					}
6590: 					$id_order_driver_status = '4';
6591: 					$datetime_prefix = 'arrive_datetime';
6592: 					$d['current_car_count']++;
```

### `archive_17012026_1259/taxi/models/api.php:6709`
```text
6707: 					$s = "UPDATE `order_driver`
6708: 						SET 
6709: 							`id_order_driver_status` = '2',
6710: 							`cancel_datetime` = now()
6711: 						WHERE
```

### `archive_17012026_1259/taxi/models/api.php:6804`
```text
6802: 				return $this->showError('404', 'error', 'unauthorized access');
6803: 			}
6804: 			if ($this->id_role != 2)
6805: 			{
6806: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:6848`
```text
6846: 				return $this->showError('404', 'error', 'not appointed performer');
6847: 			}
6848: 			elseif ($d['id_order_driver_status'] == 2 )
6849: 			{
6850: 				return $this->showError('404', 'error', 'canceled performer');
```

### `archive_17012026_1259/taxi/models/api.php:6852`
```text
6850: 				return $this->showError('404', 'error', 'canceled performer');
6851: 			}
6852: 			elseif ($d['id_order_driver_status'] == 4 )
6853: 			{
6854: 				return $this->showError('404', 'error', 'arrive state has already been changed');
```

### `archive_17012026_1259/taxi/models/api.php:6856`
```text
6854: 				return $this->showError('404', 'error', 'arrive state has already been changed');
6855: 			}
6856: 			elseif ($d['id_order_driver_status'] == 5)
6857: 			{
6858: 				return $this->showError('404', 'error', 'started ride');
```

### `archive_17012026_1259/taxi/models/api.php:6860`
```text
6858: 				return $this->showError('404', 'error', 'started ride');
6859: 			}
6860: 			elseif ($d['id_order_driver_status'] == 6)
6861: 			{
6862: 				return $this->showError('404', 'error', 'completed booking');
```

### `archive_17012026_1259/taxi/models/api.php:6869`
```text
6867: 					`order`.`last_edit_datetime` = now(),
6868: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6869: 					`order_driver`.`id_order_driver_status` = '4',
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
```

### `archive_17012026_1259/taxi/models/api.php:6901`
```text
6899: 				return $this->showError('404', 'error', 'unauthorized access');
6900: 			}
6901: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6902: 			{
6903: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:6906`
```text
6904: 			}
6905: 
6906: 			if ($this->id_role == 1 || $this->id_role == 5)
6907: 			{
6908: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:6949`
```text
6947: 				}
6948: 			}
6949: 			elseif ($this->id_role == 2)
6950: 			{		
6951: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:6998`
```text
6996: 							`order`.`last_edit_datetime` = now(),
6997: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6998: 							`order_driver`.`id_order_driver_status` = '6',
6999: 							`order_driver`.`complete_datetime` = now()
7000: 						WHERE
```

### `archive_17012026_1259/taxi/models/api.php:7055`
```text
7053: 				return $this->showError('404', 'error', 'unauthorized access');
7054: 			}
7055: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5 && $this->id_role != 4)
7056: 			{
7057: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:7082`
```text
7080: 			}
7081: 	
7082: 			if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
7083: 			{
7084: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:7106`
```text
7104: 					return $this->showError('404', 'error', 'wrong booking state');
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
7107: 				{
7108: 					return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:7239`
```text
7237: 							`order`.`last_edit_datetime` = now(),
7238: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7239: 							`order_driver`.`id_order_driver_status` = '2',
7240: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7241: 							`order_driver`.`cancel_datetime` = now(),
```

### `archive_17012026_1259/taxi/models/api.php:7265`
```text
7263: 					if ($q === false) return $this->showError('404', 'error', 'commit query failed');
7264: 				}
7265: 				elseif ($d['id_order_driver_status'] == 2)
7266: 				{
7267: 					return $this->showError('404', 'error', 'canceled performer');
```

### `archive_17012026_1259/taxi/models/api.php:7269`
```text
7267: 					return $this->showError('404', 'error', 'canceled performer');
7268: 				}
7269: 				elseif ($d['id_order_driver_status'] == 3 || $d['id_order_driver_status'] == 4)
7270: 				{
7271: 					$q = query("BEGIN");
```

### `archive_17012026_1259/taxi/models/api.php:7296`
```text
7294: 								) < `order`.`car_count`+1,
7295: 								IF(`order`.`id_order_status` in (1,5,6),`order`.`id_order_status`,1),2),
7296: 							`order_driver`.`id_order_driver_status` = '2',
7297: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7298: 							`order_driver`.`cancel_datetime` = now(),
```

### `archive_17012026_1259/taxi/models/api.php:7340`
```text
7338: 					if ($q === false) return $this->showError('404', 'error', 'commit query failed');
7339: 				}
7340: 				elseif ($d['id_order_driver_status'] == 5)
7341: 				{
7342: 					$q = query("BEGIN");
```

### `archive_17012026_1259/taxi/models/api.php:7357`
```text
7355: 							`order`.`id_order_status` = '3',
7356: 							`order`.`cancel_datetime` = now(),
7357: 							`order_driver`.`id_order_driver_status` = '2',
7358: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7359: 							`order_driver`.`cancel_datetime` = now(),
```

### `archive_17012026_1259/taxi/models/api.php:7367`
```text
7365: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7366: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
7367: 							`order_driver`.`id_order_driver_status` = '5'
7368: 						";
7369: 						
```

### `archive_17012026_1259/taxi/models/api.php:7384`
```text
7382: 					if ($q === false) return $this->showError('404', 'error', 'commit query failed');
7383: 				}
7384: 				elseif ($d['id_order_driver_status'] == 6)
7385: 				{
7386: 					return $this->showError('404', 'error', 'completed booking');
```

### `archive_17012026_1259/taxi/models/api.php:7401`
```text
7399: 				return $this->showError('404', 'error', 'unauthorized access');
7400: 			}
7401: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
7402: 			{
7403: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:7406`
```text
7404: 			}
7405: 
7406: 			if ($this->id_role == 1 || $this->id_role == 5)
7407: 			{
7408: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:7489`
```text
7487: 				}
7488: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
7489: 				if ($d['id_order_driver_status'] == 1 || $d['id_order_driver_status'] == 2) 
7490: 				{
7491: 					return $this->showError('404', 'error', 'wrong booking driver state');
```

### `archive_17012026_1259/taxi/models/api.php:7531`
```text
7529: 				return $this->showError('404', 'error', 'unauthorized access');
7530: 			}
7531: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
7532: 			{
7533: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:7594`
```text
7592: 			}
7593: 
7594: 			if ($this->id_role == 1 || $this->id_role == 4 || $this->id_role == 5)
7595: 			{
7596: 				$sql_order .= "
```

### `archive_17012026_1259/taxi/models/api.php:7618`
```text
7616: 				$sql_c_options .= "`order_driver`.`options`";
7617: 			}
7618: 			elseif ($this->id_role == 2)
7619: 			{
7620: 				$sql_order .= "
```

### `archive_17012026_1259/taxi/models/api.php:7639`
```text
7637: 			}
7638: 
7639: 			if ($this->id_role == 2 || $this->id_role == 4)
7640: 			{
7641: 				$sql_order_driver .= "
```

### `archive_17012026_1259/taxi/models/api.php:7649`
```text
7647: 			}
7648: 
7649: 			if ($this->id_role == 2)
7650: 			{
7651: 				$sql_left_join .= "
```

### `archive_17012026_1259/taxi/models/api.php:7678`
```text
7676: 					) IS NOT NULL") . ") AND `order`.`client` != '" . $_SESSION[UID] . "'";	
7677: 			}
7678: 			elseif ($this->id_role == 1 || $this->id_role == 5)
7679: 			{
7680: 				$sql_where .= " AND `order`.`client` = '" . $_SESSION[UID] . "'";
```

### `archive_17012026_1259/taxi/models/api.php:7875`
```text
7873: 					{
7874: 						$d['drivers'][$key] = array();
7875: 						if ($this->id_role == 1 || $this->id_role == 5)
7876: 						{
7877: 							list(
```

### `archive_17012026_1259/taxi/models/api.php:7928`
```text
7926: 									$d['drivers'][$key]['c_payment_sum'] = NULL;
7927: 								}
7928: 								if ($this->id_role != 4 && $_SESSION[UID] != $d['drivers'][$key]['u_id'])
7929: 								{
7930: 									unset($d['drivers'][$key]['c_payment_way']);
```

### `archive_17012026_1259/taxi/models/api.php:7948`
```text
7946: 							$d['drivers'][$key]['c_cancel_reason'] = NULL;
7947: 						}
7948: 						if ($d['drivers'][$key]['c_rating'] === chr(2))
7949: 						{
7950: 							$d['drivers'][$key]['c_rating'] = NULL;
```

### `archive_17012026_1259/taxi/models/api.php:7952`
```text
7950: 							$d['drivers'][$key]['c_rating'] = NULL;
7951: 						}
7952: 						if ($d['drivers'][$key]['c_tips'] === chr(2))
7953: 						{
7954: 							$d['drivers'][$key]['c_tips'] = NULL;
```

### `archive_17012026_1259/taxi/models/api.php:7974`
```text
7972: 						if (!empty($d['drivers'][$key]['c_options'])) $d['drivers'][$key]['c_options'] = json_decode($d['drivers'][$key]['c_options'],true);	
7973: 						
7974: 						if ($this->id_role == 4)
7975: 						{
7976: 							add_user_for_order($users_driver[$d['drivers'][$key]['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:7978`
```text
7976: 							add_user_for_order($users_driver[$d['drivers'][$key]['u_id']],'max');
7977: 						}
7978: 						elseif ($this->id_role == 1 || $this->id_role == 5)
7979: 						{
7980: 							if ((int)$d['drivers'][$key]['c_state'] > 1)
```

### `archive_17012026_1259/taxi/models/api.php:8020`
```text
8018: 					}
8019: 				}
8020: 				if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
8021: 				{
8022: 					add_user_for_order($users_client[$d['u_id']],'max');
```

### `archive_17012026_1259/taxi/models/api.php:8055`
```text
8053: 
8054: 				$d['b_options'] = json_decode($d['b_options'],true);
8055: 				if ($this->id_role == 2 && is_array($d['b_options']))
8056: 				{
8057: 					unset($d['b_options'][':u_id_alias']);
```

### `archive_17012026_1259/taxi/models/api.php:8069`
```text
8067: 				add_time_zone($d['b_start_datetime']);
8068: 				add_time_zone($d['b_created']);
8069: 				if ($this->id_role == 1 || $this->id_role == 4 || $this->id_role == 5)
8070: 				{
8071: 					add_time_zone($d['b_confirmation_limit']);
```

### `archive_17012026_1259/taxi/models/api.php:8186`
```text
8184: 				return $this->showError('404', 'error', 'unauthorized access');
8185: 			}
8186: 			if ($this->id_role != 1 && $this->id_role != 5)
8187: 			{
8188: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:8253`
```text
8251: 				return $this->showError('404', 'error', 'unauthorized access');
8252: 			}
8253: 			if ($this->id_role != 2)
8254: 			{
8255: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:8297`
```text
8295: 				return $this->showError('404', 'error', 'not appointed performer');
8296: 			}
8297: 			elseif ($d['id_order_driver_status'] == 2)
8298: 			{
8299: 				return $this->showError('404', 'error', 'canceled performer');
```

### `archive_17012026_1259/taxi/models/api.php:8301`
```text
8299: 				return $this->showError('404', 'error', 'canceled performer');
8300: 			}
8301: 			elseif ($d['id_order_driver_status'] == 5)
8302: 			{
8303: 				return $this->showError('404', 'error', 'already started ride');
```

### `archive_17012026_1259/taxi/models/api.php:8305`
```text
8303: 				return $this->showError('404', 'error', 'already started ride');
8304: 			}
8305: 			elseif ($d['id_order_driver_status'] == 6)
8306: 			{
8307: 				return $this->showError('404', 'error', 'completed booking');
```

### `archive_17012026_1259/taxi/models/api.php:8314`
```text
8312: 					`order`.`last_edit_datetime` = now(),
8313: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
8314: 					`order_driver`.`id_order_driver_status` = '5',
8315: 					`order_driver`.`start_datetime` = now()
8316: 				WHERE
```

### `archive_17012026_1259/taxi/models/api.php:8744`
```text
8742: 				return $this->showError('404', 'error', 'unauthorized access');
8743: 			}
8744: 			if ($this->id_role != 1 && $this->id_role != 5)
8745: 			{
8746: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:8972`
```text
8970: 				return $this->showError('404', 'error', 'unauthorized access');
8971: 			}
8972: 			if ($this->id_role != 2)
8973: 			{
8974: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:9256`
```text
9254: 			}
9255: 
9256: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
9257: 			{
9258: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:9312`
```text
9310: 			$user_service_profile = NULL;
9311: 
9312: 			if ($this->id_role == 1 || $this->id_role == 5)
9313: 			{
9314: 
```

### `archive_17012026_1259/taxi/models/api.php:9319`
```text
9317: 						`order`.`client`,
9318: 						`order`.`id_order_status`,
9319: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9320: 						`order`.`night_time`,
9321: 						`order`.`distance_estimate`,
```

### `archive_17012026_1259/taxi/models/api.php:9946`
```text
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
```

### `archive_17012026_1259/taxi/models/api.php:10662`
```text
10660: 			{
10661: 				$id_user = $_SESSION[UID];
10662: 				if ($this->id_role != 4) $sql_add = "";
10663: 			}
10664: 			else
```

### `archive_17012026_1259/taxi/models/api.php:10666`
```text
10664: 			else
10665: 			{	
10666: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10667: 			}
10668: 
```

### `archive_17012026_1259/taxi/models/api.php:10804`
```text
10802: 			else
10803: 			{	
10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10805: 				$s = "SELECT
10806: 						`id_user`
```

### `archive_17012026_1259/taxi/models/api.php:10891`
```text
10889: 			else
10890: 			{	
10891: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10892: 				$s = "SELECT
10893: 						`id_user`
```

### `archive_17012026_1259/taxi/models/api.php:10935`
```text
10933: 			else
10934: 			{	
10935: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10936: 
10937: 				$sql_where = "AND `id_user` in (" . $id_user . ")";
```

### `archive_17012026_1259/taxi/models/api.php:11086`
```text
11084: 			{
11085: 				$id_user = $_SESSION[UID];
11086: 				if ($this->id_role != 4) $sql_add = "";
11087: 			}
11088: 			else
```

### `archive_17012026_1259/taxi/models/api.php:11090`
```text
11088: 			else
11089: 			{	
11090: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11091: 			}
11092: 
```

### `archive_17012026_1259/taxi/models/api.php:11206`
```text
11204: 				return $this->showError('404', 'error', 'unauthorized access');
11205: 			}
11206: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
11207: 			{
11208: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:11211`
```text
11209: 			}
11210: 
11211: 			if ($this->id_role == 1 || $this->id_role == 5)
11212: 			{
11213: 				if ($b_tips === NULL) return $this->showError('404', 'error', 'null b_tips');
```

### `archive_17012026_1259/taxi/models/api.php:11302`
```text
11300: 				if ($d['tips'] !== NULL) return $this->showError('404', 'error', 'b_tips already inputed');
11301: 
11302: 				if ($d['id_order_driver_status'] == 1 || $d['id_order_driver_status'] == 2) 
11303: 				{
11304: 					return $this->showError('404', 'error', 'wrong booking driver state');
```

### `archive_17012026_1259/taxi/models/api.php:11373`
```text
11371: 			if (empty($id_user) || $id_user == $_SESSION[UID])
11372: 			{
11373: 				if ($this->id_role != 2)
11374: 				{
11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:11375`
```text
11373: 				if ($this->id_role != 2)
11374: 				{
11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
11376: 				}
11377: 				else
```

### `archive_17012026_1259/taxi/models/api.php:11392`
```text
11390: 			else
11391: 			{
11392: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11393: 				$s = "SELECT 
11394: 						`id_user`
```

### `archive_17012026_1259/taxi/models/api.php:12137`
```text
12135: 			else
12136: 			{
12137: 				if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
12138: 				{
12139: 					if ($type !== NULL) return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:12229`
```text
12227: 				$sql_where .= " AND `order_trip`.`id_order` in (" . $id_order . ")";
12228: 			}
12229: 			if ($this->id_role == 2 || $this->id_role == 4)
12230: 			{
12231: 				$orders = ",
```

### `archive_17012026_1259/taxi/models/api.php:12242`
```text
12240: 					) as orders";
12241: 				$sql_left_join = "LEFT JOIN `order_trip` USING (`id_trip`)";
12242: 				if ($this->id_role == 2) $sql_where .= " AND `trip`.`driver` = '" . $_SESSION[UID] . "'";
12243: 			}
12244: 			elseif ($this->id_role == 1 || $this->id_role == 5)
```

### `archive_17012026_1259/taxi/models/api.php:12244`
```text
12242: 				if ($this->id_role == 2) $sql_where .= " AND `trip`.`driver` = '" . $_SESSION[UID] . "'";
12243: 			}
12244: 			elseif ($this->id_role == 1 || $this->id_role == 5)
12245: 			{
12246: 				$sql_where .= " AND `trip`.`driver` != '" . $_SESSION[UID] . "'";
```

### `archive_17012026_1259/taxi/models/api.php:12250`
```text
12248: 			if ($type == 'active')
12249: 			{
12250: 				if ($this->id_role == 1 || $this->id_role == 5)
12251: 				{
12252: 					$orders = ",
```

### `archive_17012026_1259/taxi/models/api.php:12281`
```text
12279: 			elseif ($type == 'processing')
12280: 			{
12281: 				if ($this->id_role == 1 || $this->id_role == 5)
12282: 				{
12283: 					$sql_left_join = "LEFT JOIN (
```

### `archive_17012026_1259/taxi/models/api.php:12302`
```text
12300: 			else
12301: 			{
12302: 				if ($this->id_role == 1 || $this->id_role == 5)
12303: 				{
12304: 					$orders = ",
```

### `archive_17012026_1259/taxi/models/api.php:12498`
```text
12496: 							}
12497: 							if ($d['sold_seats'][$key]['number'] === chr(2)) $d['sold_seats'][$key]['number'] = NULL;
12498: 							if ($d['sold_seats'][$key]['client'] === chr(2))
12499: 							{
12500: 								switch($d['sold_seats'][$key]['status'])
```

### `archive_17012026_1259/taxi/models/api.php:12592`
```text
12590: 								'z' => ''
12591: 						);						
12592: 						if ($this->id_role != 2 && $this->id_role != 4)
12593: 						{
12594: 							foreach($d['t_options']['seats_sold'] as $seat=>$val)
```

### `archive_17012026_1259/taxi/models/api.php:12886`
```text
12884: 			}
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:12982`
```text
12980: 			$d = fetch_assoc($q);
12981: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
12982: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
12983: 			{
12984: 				return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:13005`
```text
13003: 						)= explode(chr(0),$value);
13004: 
13005: 					if ($d['orders'][$key]['id_order_driver_status'] === chr(2))
13006: 					{
13007: 						$d['orders'][$key]['id_order_driver_status'] = NULL;
```

### `archive_17012026_1259/taxi/models/api.php:13572`
```text
13570: 				return $this->showError('404', 'error', 'unauthorized access');
13571: 			}
13572: 			if ($this->id_role != 1 && $this->id_role != 5)
13573: 			{
13574: 				return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:13730`
```text
13728: 				if ($id_user != $_SESSION[UID])
13729: 				{
13730: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
13731: 
13732: 					$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:13751`
```text
13749: 			{
13750: 				$id_dropbox_link = $file['dl_id'];
13751: 				if ($this->id_role == 4)
13752: 				{
13753: 					$sql = '';
```

### `archive_17012026_1259/taxi/models/api.php:14010`
```text
14008: 					";
14009: 			}
14010: 			elseif ($this->id_role == 4)
14011: 			{
14012: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:14346`
```text
14344: 			}
14345: 
14346: 			if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
14347: 			{
14348: 				return $this->showError('404', 'error', 'wrong user check state');
```

### `archive_17012026_1259/taxi/models/api.php:16850`
```text
16848: 			if (empty($s_date['site_constants']['data_edit_users_roles_rights']['value']) || empty($data_edit_users_roles_rights = json_decode($s_date['site_constants']['data_edit_users_roles_rights']['value'],true)) || !is_array($data_edit_users_roles_rights))
16849: 			{
16850: 				if ($this->id_role != 4)
16851: 				{
16852: 					return $this->showError('404', 'error', 'not enough rights');			
```

### `archive_17012026_1259/taxi/models/api.php:17403`
```text
17401: 				return $this->showError('404', 'error', 'unauthorized access');
17402: 			}
17403: 			if ($this->id_role != 4)
17404: 			{
17405: 				return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:17449`
```text
17447: 			if ($filter == 'all')
17448: 			{
17449: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
17450: 				$sql_where = "1 = 1";
17451: 			}
```

### `archive_17012026_1259/taxi/models/api.php:17454`
```text
17452: 			elseif ($filter == 'trip')	
17453: 			{
17454: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter');
17455: 				$sql_where = "`trip`.`driver` = '" . $_SESSION[UID] . "'";
17456: 			}
```

### `archive_17012026_1259/taxi/models/api.php:17892`
```text
17890: 			if (!empty($query_roles))
17891: 			{
17892: 				if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
17893: 				{
17894: 					return $this->showError('404', 'error', 'wrong user check state');
```

### `archive_17012026_1259/taxi/models/api.php:17898`
```text
17896: 				$query_roles = explode(',',$query_roles);
17897: 				$query_roles = array_flip($query_roles);
17898: 				if (!isset($query_roles[$this->id_role])) return $this->showError('404', 'error', 'forbidden role');
17899: 			}
17900: 
```

### `archive_17012026_1259/taxi/models/api.php:17911`
```text
17909: 				elseif ($statement == 'custom')
17910: 				{
17911: 					if ($this->id_role != 4)  return $this->showError('404', 'error', 'not enough rights');
17912: 					if ($hash !=  md5('checking' . md5(API_KEY))) return $this->showError('404', 'error', 'wrong hash');
17913: 					
```

### `archive_17012026_1259/taxi/models/api.php:18136`
```text
18134: 			$d = fetch_assoc($q);
18135: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18136: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18137: 			{
18138: 				return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:18260`
```text
18258: 				$d = fetch_assoc($q);
18259: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18260: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18261: 				{
18262: 					return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:18368`
```text
18366: 				{
18367: 					if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
18368: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18369: 					{
18370: 						if (empty($d['client']) || $d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller or buyer');
```

### `archive_17012026_1259/taxi/models/api.php:18377`
```text
18375: 				{
18376: 					if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
18377: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18378: 					{
18379: 						if (!array($t_options['seats_sold'][$seat]) || count($t_options['seats_sold'][$seat]) < 2 || $t_options['seats_sold'][$seat][1] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller and buyer');
```

### `archive_17012026_1259/taxi/models/api.php:18624`
```text
18622: 			if ($filter == 'all')
18623: 			{
18624: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter all');
18625: 				$sql_field = "`cart_block`.`id_user` as u_id,
18626: 				`cart_block`.`active`,";
```

### `archive_17012026_1259/taxi/models/api.php:18627`
```text
18625: 				$sql_field = "`cart_block`.`id_user` as u_id,
18626: 				`cart_block`.`active`,";
18627: 				if ($this->id_role != 4)
18628: 				{
18629: 					$sql_group_concat = "IF(`trip`.`driver` = '{$_SESSION[UID]}',
```

### `archive_17012026_1259/taxi/models/api.php:18641`
```text
18639: 			elseif ($filter == 'trip')
18640: 			{
18641: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter trip');
18642: 				$sql_field = "`cart_block`.`id_user` as u_id,
18643: 				`cart_block`.`active`,";
```

### `archive_17012026_1259/taxi/models/api.php:18644`
```text
18642: 				$sql_field = "`cart_block`.`id_user` as u_id,
18643: 				`cart_block`.`active`,";
18644: 				if ($this->id_role != 4)
18645: 				{
18646: 					$sql_group_concat = "IF(`trip`.`driver` = '{$_SESSION[UID]}',
```

### `archive_17012026_1259/taxi/models/api.php:18958`
```text
18956: 				";
18957: 
18958: 			if (empty($_SESSION[UID]) || $this->id_role != 2) {
18959: 				if (empty($sc_id)) return $this->showError('404', 'error', 'empty sc_id');
18960: 			}
```

### `archive_17012026_1259/taxi/models/api.php:19126`
```text
19124: 			{
19125: 				if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
19126: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller');
19127: 				if ($ignore_order === false && empty($d['client'])) return $this->showError('404', 'error', 'buyer not found');				
19128: 				if ($ignore_order === false && $d['pay_datetime'] == '0000-00-00 00:00:00') return $this->showError('404', 'error', 'unpaid order');
```

### `archive_17012026_1259/taxi/models/api.php:19127`
```text
19125: 				if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
19126: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller');
19127: 				if ($ignore_order === false && empty($d['client'])) return $this->showError('404', 'error', 'buyer not found');				
19128: 				if ($ignore_order === false && $d['pay_datetime'] == '0000-00-00 00:00:00') return $this->showError('404', 'error', 'unpaid order');
19129: 				if (empty($email) && empty($id_user))
```

### `archive_17012026_1259/taxi/models/api.php:19138`
```text
19136: 			{
19137: 				if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
19138: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller of trip');
19139: 				if (!array($t_options['seats_sold'][$seat]) || count($t_options['seats_sold'][$seat]) < 2) 
19140: 				{
```

### `archive_17012026_1259/taxi/models/api.php:19173`
```text
19171: 			if ($resend === true)
19172: 			{
19173: 				if (empty($d['client'])) return $this->showError('404', 'error', 'ticket without buyer');	
19174: 				$api_use = true;
19175: 				$order = $d['id_order'];
```

### `archive_17012026_1259/taxi/models/api.php:19814`
```text
19812: 			}
19813: 			$is_usher = false;
19814: 			if ($this->id_role != 4 && ($d['driver'] != $_SESSION[UID] || $this->id_role == 6)) 
19815: 			{
19816: 				if ($this->id_role == 6)
```

### `archive_17012026_1259/taxi/models/api.php:19816`
```text
19814: 			if ($this->id_role != 4 && ($d['driver'] != $_SESSION[UID] || $this->id_role == 6)) 
19815: 			{
19816: 				if ($this->id_role == 6)
19817: 				{
19818: 				
```

### `archive_17012026_1259/taxi/models/api.php:19862`
```text
19860: 					}
19861: 				}
19862: 				elseif ($this->id_role != 4)
19863: 				{
19864: 					unset($arr['pass']);
```

### `archive_17012026_1259/taxi/models/api.php:20153`
```text
20151: 			if (!isset($sql_templates[$template])) return $this->showError('404', 'error', 'template not found');
20152: 
20153: 			if (!empty($sql_templates[$template]['only_admin']) && $this->id_role != 4)
20154: 			{
20155: 				return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:20284`
```text
20282: 			}
20283: 
20284: 			if (!empty($script_template['only_admin']) && $this->id_role != 4)
20285: 			{
20286: 				return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:20306`
```text
20304: 				if (empty($_SESSION['edit_langs'])) return $this->showError('404', 'error', 'unauthorized access');
20305: 			} else {
20306: 				if ($this->id_role != 4)
20307: 				{
20308: 					return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:20347`
```text
20345: 			}
20346: 
20347: 			if ($this->id_role != 4)
20348: 			{
20349: 				if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong user role');
```

### `archive_17012026_1259/taxi/models/api.php:20349`
```text
20347: 			if ($this->id_role != 4)
20348: 			{
20349: 				if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong user role');
20350: 				if ($_SESSION['id_verification_status'] != 2)
20351: 				{
```

### `archive_17012026_1259/taxi/models/api.php:20413`
```text
20411: 				if (!empty($filtered_data)) 
20412: 				{
20413: 					if ($this->id_role != 4) {$out['warning'][]  = "$i update wrong user role"; continue;}
20414: 	
20415: 					$s = array();
```

### `archive_17012026_1259/taxi/models/api.php:20463`
```text
20461: 							}
20462: 							$driver = $d['driver'];
20463: 							if ($driver != $_SESSION[UID] && $this->id_role != 4)
20464: 							{
20465: 								$out['warning'][]  = "$i $j foreign trip";
```

### `archive_17012026_1259/taxi/models/api.php:20523`
```text
20521: 			{
20522: 				$id_user = $_SESSION[UID];
20523: 				if ($this->id_role != 4) $sql_add = "";
20524: 			}
20525: 			else
```

### `archive_17012026_1259/taxi/models/api.php:20527`
```text
20525: 			else
20526: 			{	
20527: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
20528: 			}
20529: 
```

### `archive_17012026_1259/taxi/models/api.php:20674`
```text
20672: 					break;
20673: 				case '1':
20674: 					if ($this->id_role != 4)
20675: 					{
20676: 						return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:20866`
```text
20864: 						`contact_items`.`smtpsecure`,
20865: 						`contact_items`.`fromname`";
20866: 			if ($this->id_role != 4) $sql_private_o_type_other = ",NULL as 'number',
20867: 						NULL as 'key1',
20868: 						NULL as 'key2',
```

### `archive_17012026_1259/taxi/models/api.php:20880`
```text
20878: 			if ($id_user === NULL)
20879: 			{
20880: 				if ($this->id_role != 4) $id_user = $_SESSION[UID];
20881: 			}
20882: 			else
```

### `archive_17012026_1259/taxi/models/api.php:20884`
```text
20882: 			else
20883: 			{
20884: 				if ($this->id_role != 4 && $id_user != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
20885: 			}
20886: 			$sql_name = $sql_description = array();
```

### `archive_17012026_1259/taxi/models/api.php:21264`
```text
21262: 				if (isset($filtered_data['id_owner_type']) && $filtered_data['id_owner_type'] != 1) 
21263: 				{
21264: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
21265: 				}
21266: 				else
```

### `archive_17012026_1259/taxi/models/api.php:21268`
```text
21266: 				else
21267: 				{
21268: 					if ($filtered_data['owner'] != $_SESSION[UID] && $this->id_role != 4) return $this->showError('404', 'error', 'not enough rights for action');
21269: 				}
21270: 			}
```

### `archive_17012026_1259/taxi/models/api.php:21446`
```text
21444: 			$d = fetch_assoc($q);
21445: 			if (empty($d['id_contact_item'])) return $this->showError('404', 'error', 'contact not found');
21446: 			if ($this->id_role != 4 && ($d['id_owner_type'] != 1 || $d['owner'] != $_SESSION[UID])) return $this->showError('404', 'error', 'not enough rights');
21447: 
21448: 			if ($this->id_role != 4)
```

### `archive_17012026_1259/taxi/models/api.php:21448`
```text
21446: 			if ($this->id_role != 4 && ($d['id_owner_type'] != 1 || $d['owner'] != $_SESSION[UID])) return $this->showError('404', 'error', 'not enough rights');
21447: 
21448: 			if ($this->id_role != 4)
21449: 			{
21450: 				unset($allowed_fields['owner']);
```

### `archive_17012026_1259/taxi/models/api.php:21838`
```text
21836: 					$from_owner = $contacts[$co_id]['owner'];
21837: 					$from_o_type = $contacts[$co_id]['id_owner_type'];
21838: 					if ($this->id_role != 4 && $from_o_type != 1) return $this->showError('404', 'error', 'code: not enough rights');
21839: 					$owner_types_filtered = array($from_o_type => $owner_types[$from_o_type]);
21840: 				}
```

### `archive_17012026_1259/taxi/models/api.php:22616`
```text
22614: 			}
22615: 
22616: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22617: 
22618: 			if (empty($data)) 
```

### `archive_17012026_1259/taxi/models/api.php:22901`
```text
22899: 			}
22900: 
22901: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22902: 
22903: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
```

### `archive_17012026_1259/taxi/models/api.php:22932`
```text
22930: 			}
22931: 
22932: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22933: 
22934: 			$sql_where = '1 = 1';
```

### `archive_17012026_1259/taxi/models/api.php:23051`
```text
23049: 			}
23050: 
23051: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23052: 
23053: 			if (empty($data)) 
```

### `archive_17012026_1259/taxi/models/api.php:23184`
```text
23182: 			}
23183: 
23184: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23185: 
23186: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
```

### `archive_17012026_1259/taxi/models/api.php:23239`
```text
23237: 			}
23238: 
23239: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23240: 
23241: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
```

### `archive_17012026_1259/taxi/models/api.php:23390`
```text
23388: 			if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link not found');
23389: 			if (!empty($d['deleted'])) return $this->showError('404', 'error', 'dropbox file already deleted');
23390: 			if ($this->id_role != 4 && $d['user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
23391: 			$json = json_decode($d['json'],true);
23392: 			
```

### `archive_17012026_1259/taxi/models/api.php:23473`
```text
23471: 				if (empty($this->associativeArray)) $out['contacts'] =  array_values($out['contacts']);
23472: 			}
23473: 			elseif ($this->id_role == 4)
23474: 			{
23475: 				$sql_where = '1 = 1';
```

### `archive_17012026_1259/taxi/models/api.php:23617`
```text
23615: 
23616: 			$sql_where_add = " AND (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NULL AND `ticket`.`status` not in (2,3))";
23617: 			if ($taken == true && ($this->id_role == 1 || $this->id_role == 2 || $this->id_role == 4))
23618: 			{
23619: 				if ($this->id_role == 4)
```

### `archive_17012026_1259/taxi/models/api.php:23619`
```text
23617: 			if ($taken == true && ($this->id_role == 1 || $this->id_role == 2 || $this->id_role == 4))
23618: 			{
23619: 				if ($this->id_role == 4)
23620: 				{
23621: 					$sql_field = $sql_field_code_qr;
```

### `archive_17012026_1259/taxi/models/api.php:23639`
```text
23637: 					$sql_left_join = "LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`";
23638: 				}
23639: 				elseif ($this->id_role == 2)
23640: 				{
23641: 					$sql_field = $sql_field_code_qr;
```

### `archive_17012026_1259/taxi/models/api.php:23650`
```text
23648: 			else
23649: 			{
23650: 				if ($this->id_role == 4)
23651: 				{
23652: 					$sql_field = $sql_field_code_qr;
```

### `archive_17012026_1259/taxi/models/api.php:23670`
```text
23668: 					$sql_left_join = "LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`";
23669: 				}
23670: 				elseif ($this->id_role == 2)
23671: 				{
23672: 					$sql_field = $sql_field_code_qr;
```

### `archive_17012026_1259/taxi/models/api.php:24234`
```text
24232: 				return $this->showError('404', 'error', 'empty code');
24233: 			}
24234: 			if ($this->id_role != 4 && $this->id_role != 6) 
24235: 			{
24236: 				return $this->showError('404', 'error', 'wrong role');
```

### `archive_17012026_1259/taxi/models/api.php:24245`
```text
24243: 			$sql_where = "`ticket`.`code` = '$code'";
24244: 
24245: 			if ($this->id_role == 6)
24246: 			{
24247: 				$sql_field = '';
```

### `archive_17012026_1259/taxi/models/api.php:24456`
```text
24454: 			}
24455: 
24456: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24457: 			
24458: 			$sql_where = array();
```

### `archive_17012026_1259/taxi/models/api.php:24598`
```text
24596: 						COUNT(`id_order_driver_status`) as 'all',
24597: 						COUNT(IF(`id_order_driver_status` = 1,1,NULL)) as candidacy,
24598: 						COUNT(IF(`id_order_driver_status` = 2,1,NULL)) as canceled,
24599: 						COUNT(IF(`id_order_driver_status` = 3,1,NULL)) as appointed,
24600: 						COUNT(IF(`id_order_driver_status` = 4,1,NULL)) as arrived,
```

### `archive_17012026_1259/taxi/models/api.php:24600`
```text
24598: 						COUNT(IF(`id_order_driver_status` = 2,1,NULL)) as canceled,
24599: 						COUNT(IF(`id_order_driver_status` = 3,1,NULL)) as appointed,
24600: 						COUNT(IF(`id_order_driver_status` = 4,1,NULL)) as arrived,
24601: 						COUNT(IF(`id_order_driver_status` = 5,1,NULL)) as started,
24602: 						COUNT(IF(`id_order_driver_status` = 6,1,NULL)) as completed
```

### `archive_17012026_1259/taxi/models/api.php:24601`
```text
24599: 						COUNT(IF(`id_order_driver_status` = 3,1,NULL)) as appointed,
24600: 						COUNT(IF(`id_order_driver_status` = 4,1,NULL)) as arrived,
24601: 						COUNT(IF(`id_order_driver_status` = 5,1,NULL)) as started,
24602: 						COUNT(IF(`id_order_driver_status` = 6,1,NULL)) as completed
24603: 					FROM `order_driver`
```

### `archive_17012026_1259/taxi/models/api.php:24602`
```text
24600: 						COUNT(IF(`id_order_driver_status` = 4,1,NULL)) as arrived,
24601: 						COUNT(IF(`id_order_driver_status` = 5,1,NULL)) as started,
24602: 						COUNT(IF(`id_order_driver_status` = 6,1,NULL)) as completed
24603: 					FROM `order_driver`
24604: 					WHERE
```

### `archive_17012026_1259/taxi/models/api.php:24772`
```text
24770: 			if (array_key_exists('from',$filtered_data))
24771: 			{
24772: 				if ($this->id_role != 4) 
24773: 				{
24774: 					return $this->showError('404', 'error', 'not enough rights');
```

### `archive_17012026_1259/taxi/models/api.php:24910`
```text
24908: 				if ($d['from'] != $_SESSION[UID])
24909: 				{
24910: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24911: 				}
24912: 				$d['json'] = json_decode($d['json'],true);
```

### `archive_17012026_1259/taxi/models/api.php:25064`
```text
25062: 
25063: 			if (empty($filtered_data['id_tariff'])) return $this->showError('404', 'error', 'empty tariff');
25064: 			if ($this->id_role != 4) 
25065: 			{
25066: 				while (true)
```

### `archive_17012026_1259/taxi/models/api.php:25111`
```text
25109: 			$sql_where = "1 = 1";
25110: 			if (!empty($id_user_subscription)) $sql_where .= " AND `user_subscription`.`id_user_subscription` in ($id_user_subscription)";
25111: 			if ($this->id_role = 4) 
25112: 			{
25113: 				if (!empty($id_user)) $sql_where .= " AND `user_subscription`.`id_user` in ($id_user)";
```

### `archive_17012026_1259/taxi/models/api.php:25381`
```text
25379: 				$d = fetch_assoc($q);
25380: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'account not found');
25381: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25382: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
25383: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
```

### `archive_17012026_1259/taxi/models/api.php:25625`
```text
25623: 				$d = fetch_assoc($q);
25624: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'currency account not found');
25625: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25626: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
25627: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
```

## 3. Нормативное правило mapping

Mapping:

```text
Role ID → Business Role
```

получает `CONFIRMED` только если источник явно устанавливает соответствие, например:

```text
role 2 = Driver
```

или эквивалентное enum/config/data definition.

Если код только проверяет:

```text
id_role == 2
```

это Evidence использования Role ID, но не доказательство его business name.

## 4. Предварительный результат

В доступном source corpus числовые role IDs активно используются в authorization checks, role resolution и API logic. Однако mapping нельзя строить автоматически из названий методов или из контекста проверки.

Поэтому текущий pass разделяет:

```text
Role ID usage       → CONFIRMED
Role ID semantics   → only where explicit mapping exists
```

## 5. Текущая матрица

| Role ID | Business Role | Status | Evidence rule |
|---:|---|---|---|
| 2 | UNKNOWN unless explicit mapping found | OPEN | direct mapping required |
| 4 | UNKNOWN unless explicit mapping found | OPEN | direct mapping required |
| 5 | UNKNOWN unless explicit mapping found | OPEN | direct mapping required |
| 6 | UNKNOWN unless explicit mapping found | OPEN | direct mapping required |

## 6. Почему мы не называем роли по догадке

Например, если метод содержит:

```text
if ($this->id_role != 2 && $this->id_role != 4)
```

это доказывает permission relation для operation.

Но не доказывает:

```text
2 = Driver
4 = Admin
```

Такая дисциплина нужна, чтобы Role Permission Matrix не превратилась в красивую, но вымышленную domain model.

## 7. Что следует искать дальше

Наиболее сильные ожидаемые источники:

```text
users_roles / role tables
seed INSERTs
role constants / enum definitions
site_constant role descriptions
UI labels explicitly tied to role IDs
role configuration
documentation generated from source
```

Приоритет — source/config/data over comments or inferred names.

## 8. Gap Report

```text
G-14-01  Role ID 2 → business role     OPEN
G-14-02  Role ID 4 → business role     OPEN
G-14-03  Role ID 5 → business role     OPEN
G-14-04  Role ID 6 → business role     OPEN
G-14-05  Complete Role × Operation matrix OPEN
```

## 9. MCR

`MCR = NO CHANGE`.

Role mapping — это provenance-bearing claim, а не новый тип графовой сущности.

## 10. Следующий шаг

Не расширять общий поиск по коду. Следующий pass должен целенаправленно исследовать `users_roles`, role tables, seed/config records и явные role labels, чтобы получить хотя бы один подтверждённый mapping. После первого подтверждённого mapping можно проверить, распространяется ли та же конвенция на остальные IDs.
