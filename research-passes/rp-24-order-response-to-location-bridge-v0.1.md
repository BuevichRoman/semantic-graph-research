# Backend Semantic Graph — Research Pass 24
# Order Response → `/location` Bridge v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-23 Assignment → Position Exposure v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Research Question

> Выводится ли `u_id` для `/location` из assigned performer/order state в конкретном frontend path?

Целевая цепочка:

```text
Order response
   ↓
assigned driver/user ID
   ↓
frontend state
   ↓
/location?u_id=...
   ↓
Position
```

## 2. Поисковая стратегия

Поиск ограничен четырьмя элементами:
1. order/order_driver response fields;
2. `id_driver` / `id_user`;
3. frontend construction of `u_id`;
4. concrete `/location` invocation.

## 3. Наиболее релевантные contexts

Найдено candidate contexts: **1923**.

### `archive_17012026_1259/taxi/models/api.php:6194`
```text
6189: 			}
6190: 			
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
```

### `archive_17012026_1259/taxi/models/api.php:6195`
```text
6190: 			
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
```

### `archive_17012026_1259/taxi/models/api.php:6196`
```text
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
```

### `archive_17012026_1259/taxi/models/api.php:6197`
```text
6192: 			{
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
```

### `archive_17012026_1259/taxi/models/api.php:6198`
```text
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:6199`
```text
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
```

### `archive_17012026_1259/taxi/models/api.php:6200`
```text
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
```

### `archive_17012026_1259/taxi/models/api.php:6201`
```text
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
6209: 				$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:6202`
```text
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
6209: 				$q = query($s);
6210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

### `archive_17012026_1259/taxi/models/api.php:6203`
```text
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
6209: 				$q = query($s);
6210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6211: 				
```

### `archive_17012026_1259/taxi/models/api.php:6384`
```text
6379: 				$q = query("BEGIN");
6380: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
6381: 
6382: 				
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
```

### `archive_17012026_1259/taxi/models/api.php:6385`
```text
6380: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
6381: 
6382: 				
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
```

### `archive_17012026_1259/taxi/models/api.php:6386`
```text
6381: 
6382: 				
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
```

### `archive_17012026_1259/taxi/models/api.php:6387`
```text
6382: 				
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
```

### `archive_17012026_1259/taxi/models/api.php:6388`
```text
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
```

### `archive_17012026_1259/taxi/models/api.php:6389`
```text
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
```

### `archive_17012026_1259/taxi/models/api.php:6390`
```text
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
```

### `archive_17012026_1259/taxi/models/api.php:6391`
```text
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
```

### `archive_17012026_1259/taxi/models/api.php:6392`
```text
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
```

### `archive_17012026_1259/taxi/models/api.php:6393`
```text
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
6401: 						IFNULL(MAX(`order_driver`.`index`),0) as max_index,
```

### `archive_17012026_1259/taxi/models/api.php:6394`
```text
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
6401: 						IFNULL(MAX(`order_driver`.`index`),0) as max_index,
6402: 						(SELECT
```

### `archive_17012026_1259/taxi/models/api.php:6396`
```text
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
6401: 						IFNULL(MAX(`order_driver`.`index`),0) as max_index,
6402: 						(SELECT
6403: 							COUNT(`order_driver_attempt`.`id_order`)
6404: 						 FROM `order_driver_attempt`
```

### `archive_17012026_1259/taxi/models/api.php:6952`
```text
6947: 				}
6948: 			}
6949: 			elseif ($this->id_role == 2)
6950: 			{		
6951: 				$s = "SELECT
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
```

### `archive_17012026_1259/taxi/models/api.php:6953`
```text
6948: 			}
6949: 			elseif ($this->id_role == 2)
6950: 			{		
6951: 				$s = "SELECT
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
```

### `archive_17012026_1259/taxi/models/api.php:6954`
```text
6949: 			elseif ($this->id_role == 2)
6950: 			{		
6951: 				$s = "SELECT
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
```

### `archive_17012026_1259/taxi/models/api.php:6955`
```text
6950: 			{		
6951: 				$s = "SELECT
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
```

### `archive_17012026_1259/taxi/models/api.php:6956`
```text
6951: 				$s = "SELECT
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
6964: 					WHERE	
```

### `archive_17012026_1259/taxi/models/api.php:6957`
```text
6952: 						`order`.`id_order`,
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
6964: 					WHERE	
6965: 						`order`.`id_order` = '" . $id_order . "'
```

### `archive_17012026_1259/taxi/models/api.php:6958`
```text
6953: 						`order`.`id_order_status`,
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
6964: 					WHERE	
6965: 						`order`.`id_order` = '" . $id_order . "'
6966: 					LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:6959`
```text
6954: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6955: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6956: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6957: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6958: 						COUNT(IF(`order_driver`.`id_user` != '" . $_SESSION[UID] 
6959: 							. "' AND `order_driver`.`id_order_driver_status` in (3,4,5),
6960: 							1,NULL)) as incomplete_count
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
6964: 					WHERE	
6965: 						`order`.`id_order` = '" . $id_order . "'
6966: 					LIMIT 1
6967: 					";
```

### `archive_17012026_1259/taxi/models/api.php:9939`
```text
9934: 			}
9935: 			else
9936: 			{
9937: 			
9938: 				$s = "SELECT
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
```

### `archive_17012026_1259/taxi/models/api.php:9940`
```text
9935: 			else
9936: 			{
9937: 			
9938: 				$s = "SELECT
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
```

### `archive_17012026_1259/taxi/models/api.php:9941`
```text
9936: 			{
9937: 			
9938: 				$s = "SELECT
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
```

### `archive_17012026_1259/taxi/models/api.php:9942`
```text
9937: 			
9938: 				$s = "SELECT
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
```

### `archive_17012026_1259/taxi/models/api.php:9943`
```text
9938: 				$s = "SELECT
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
```

### `archive_17012026_1259/taxi/models/api.php:9944`
```text
9939: 						`order`.`id_order`,
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
9952: 						`order`.`to_lat`,
```

### `archive_17012026_1259/taxi/models/api.php:9945`
```text
9940: 						`order`.`client`,
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
9952: 						`order`.`to_lat`,
9953: 						`order`.`to_lng`,
```

### `archive_17012026_1259/taxi/models/api.php:9946`
```text
9941: 						`order`.`id_order_status`,
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
9952: 						`order`.`to_lat`,
9953: 						`order`.`to_lng`,
9954: 						`order`.`car_count`,
```

### `archive_17012026_1259/taxi/models/api.php:9947`
```text
9942: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
9952: 						`order`.`to_lat`,
9953: 						`order`.`to_lng`,
9954: 						`order`.`car_count`,
9955: 						`order`.`datetime_start_plan`,
```

### `archive_17012026_1259/taxi/models/api.php:9948`
```text
9943: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
9944: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
9945: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
9946: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9947: 						`order`.`night_time`,
9948: 						`order`.`distance_estimate`,
9949: 						`order`.`id_car_class`,
9950: 						`order`.`from_lat`,
9951: 						`order`.`from_lng`,
9952: 						`order`.`to_lat`,
9953: 						`order`.`to_lng`,
9954: 						`order`.`car_count`,
9955: 						`order`.`datetime_start_plan`,
9956: 						GROUP_CONCAT(`order_driver`.`id_car`) as cars,
```

### `archive_17012026_1259/taxi/models/api.php:13578`
```text
13573: 			{
13574: 				return $this->showError('404', 'error', 'wrong user role');
13575: 			}
13576: 			if ($id_user == $_SESSION[UID]) return $this->showError('404', 'error', 'trying to offer yourself');
13577: 			if (empty($id_order)) return $this->showError('404', 'error', 'empty booking');
13578: 			if (empty($id_user)) return $this->showError('404', 'error', 'empty driver');
13579: 
13580: 			$s = "SELECT
13581: 					`id_order`,
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
```

### `archive_17012026_1259/taxi/models/api.php:13584`
```text
13579: 
13580: 			$s = "SELECT
13581: 					`id_order`,
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
```

### `archive_17012026_1259/taxi/models/api.php:13585`
```text
13580: 			$s = "SELECT
13581: 					`id_order`,
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
13593: 			$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:13586`
```text
13581: 					`id_order`,
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
13593: 			$q = query($s);
13594: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

### `archive_17012026_1259/taxi/models/api.php:13587`
```text
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
13593: 			$q = query($s);
13594: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
13595: 
```

### `archive_17012026_1259/taxi/models/api.php:24590`
```text
24585: 
24586: 				
24587: 				if ($q === false) return array('error' => 'order stat select failed');
24588: 				while ($d = fetch_assoc($q))
24589: 				{
24590: 					$out[$d['u_id']]['stat']['client'] = $d;
24591: 					unset($out[$d['u_id']]['stat']['client']['u_id']);
24592: 				}
24593: 	
24594: 				$s = "SELECT
24595: 						`id_user` as u_id,
24596: 						COUNT(`id_order_driver_status`) as 'all',
24597: 						COUNT(IF(`id_order_driver_status` = 1,1,NULL)) as candidacy,
24598: 						COUNT(IF(`id_order_driver_status` = 2,1,NULL)) as canceled,
```

### `archive_17012026_1259/taxi/models/api.php:24591`
```text
24586: 				
24587: 				if ($q === false) return array('error' => 'order stat select failed');
24588: 				while ($d = fetch_assoc($q))
24589: 				{
24590: 					$out[$d['u_id']]['stat']['client'] = $d;
24591: 					unset($out[$d['u_id']]['stat']['client']['u_id']);
24592: 				}
24593: 	
24594: 				$s = "SELECT
24595: 						`id_user` as u_id,
24596: 						COUNT(`id_order_driver_status`) as 'all',
24597: 						COUNT(IF(`id_order_driver_status` = 1,1,NULL)) as candidacy,
24598: 						COUNT(IF(`id_order_driver_status` = 2,1,NULL)) as canceled,
24599: 						COUNT(IF(`id_order_driver_status` = 3,1,NULL)) as appointed,
```

### `archive_17012026_1259/taxi/models/api.php:24595`
```text
24590: 					$out[$d['u_id']]['stat']['client'] = $d;
24591: 					unset($out[$d['u_id']]['stat']['client']['u_id']);
24592: 				}
24593: 	
24594: 				$s = "SELECT
24595: 						`id_user` as u_id,
24596: 						COUNT(`id_order_driver_status`) as 'all',
24597: 						COUNT(IF(`id_order_driver_status` = 1,1,NULL)) as candidacy,
24598: 						COUNT(IF(`id_order_driver_status` = 2,1,NULL)) as canceled,
24599: 						COUNT(IF(`id_order_driver_status` = 3,1,NULL)) as appointed,
24600: 						COUNT(IF(`id_order_driver_status` = 4,1,NULL)) as arrived,
24601: 						COUNT(IF(`id_order_driver_status` = 5,1,NULL)) as started,
24602: 						COUNT(IF(`id_order_driver_status` = 6,1,NULL)) as completed
24603: 					FROM `order_driver`
```

### `archive_17012026_1259/taxi/config/system_bot.php:436`
```text
431: 			`order`.`from_lng`,
432: 			`order`.`id_car_class`,
433: 			(SELECT
434: 				GROUP_CONCAT(`users_favorite`.`id_favorite` SEPARATOR ',')
435: 			 FROM `users_favorite`
436: 			 WHERE `users_favorite`.`id_user` = `order`.`client`
437: 			) as  favorite,
438: 			(SELECT
439: 				CONCAT_WS('|',CAST(COUNT(`order_offer`.`id_order`) as char),CAST(MAX(`order_offer`.`create_datetime`) as datetime))
440: 			 FROM `order_offer`
441: 			 WHERE `order_offer`.`id_order` = `order`.`id_order`
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
```

### `archive_17012026_1259/taxi/config/system_bot.php:441`
```text
436: 			 WHERE `users_favorite`.`id_user` = `order`.`client`
437: 			) as  favorite,
438: 			(SELECT
439: 				CONCAT_WS('|',CAST(COUNT(`order_offer`.`id_order`) as char),CAST(MAX(`order_offer`.`create_datetime`) as datetime))
440: 			 FROM `order_offer`
441: 			 WHERE `order_offer`.`id_order` = `order`.`id_order`
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
```

### `archive_17012026_1259/taxi/config/system_bot.php:444`
```text
439: 				CONCAT_WS('|',CAST(COUNT(`order_offer`.`id_order`) as char),CAST(MAX(`order_offer`.`create_datetime`) as datetime))
440: 			 FROM `order_offer`
441: 			 WHERE `order_offer`.`id_order` = `order`.`id_order`
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
450: 		WHERE	
451: 			`order`.`id_order_status` = '6' AND `order`.`datetime_start_plan`
452: 		GROUP BY 
```

### `archive_17012026_1259/taxi/config/system_bot.php:445`
```text
440: 			 FROM `order_offer`
441: 			 WHERE `order_offer`.`id_order` = `order`.`id_order`
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
450: 		WHERE	
451: 			`order`.`id_order_status` = '6' AND `order`.`datetime_start_plan`
452: 		GROUP BY 
453: 			`order`.`id_order`
```

### `archive_17012026_1259/taxi/config/system_bot.php:446`
```text
441: 			 WHERE `order_offer`.`id_order` = `order`.`id_order`
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
450: 		WHERE	
451: 			`order`.`id_order_status` = '6' AND `order`.`datetime_start_plan`
452: 		GROUP BY 
453: 			`order`.`id_order`
454: 		";
```

### `archive_17012026_1259/taxi/config/system_bot.php:447`
```text
442: 			) as offer,			
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
450: 		WHERE	
451: 			`order`.`id_order_status` = '6' AND `order`.`datetime_start_plan`
452: 		GROUP BY 
453: 			`order`.`id_order`
454: 		";
455: 
```

### `archive_17012026_1259/taxi/config/system_bot.php:448`
```text
443: 			COUNT(IF(`order_driver_select`.`cancel` = '0',1,NULL)) as uncancelled_drivers,
444: 			GROUP_CONCAT(`order_driver_select`.`id_user` SEPARATOR ',') as already_selected,
445: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
446: 			`order`.`create_datetime`,
447: 			`order`.`car_count`
448: 		FROM `order` 
449: 		LEFT JOIN `order_driver_select` USING (`id_order`)	
450: 		WHERE	
451: 			`order`.`id_order_status` = '6' AND `order`.`datetime_start_plan`
452: 		GROUP BY 
453: 			`order`.`id_order`
454: 		";
455: 
456: 	$q = query($s);
```

### `archive_17012026_1259/taxi/config/system_bot.php:578`
```text
573: 				(
574: 					SELECT 
575: 						`id_user`
576: 					FROM `users_ban`
577: 					WHERE
578: 						`users_ban`.`id_user` = `users`.`id_user` AND (`expire_datetime` = 0 OR `expire_datetime` > NOW()) AND
579: 						(`auth` = '1' OR `order` = '1')
580: 					LIMIT 1
581: 				) IS NULL AND
582: 				(
583: 					SELECT 
584: 						`order`.`id_order`				
585: 					FROM `order`
586: 					LEFT JOIN `order_driver` USING (`id_order`)
```

### `archive_17012026_1259/taxi/config/system_bot.php:579`
```text
574: 					SELECT 
575: 						`id_user`
576: 					FROM `users_ban`
577: 					WHERE
578: 						`users_ban`.`id_user` = `users`.`id_user` AND (`expire_datetime` = 0 OR `expire_datetime` > NOW()) AND
579: 						(`auth` = '1' OR `order` = '1')
580: 					LIMIT 1
581: 				) IS NULL AND
582: 				(
583: 					SELECT 
584: 						`order`.`id_order`				
585: 					FROM `order`
586: 					LEFT JOIN `order_driver` USING (`id_order`)
587: 					LEFT JOIN `order_driver_select` USING (`id_order`)
```

### `archive_17012026_1259/taxi/config/system_bot.php:589`
```text
584: 						`order`.`id_order`				
585: 					FROM `order`
586: 					LEFT JOIN `order_driver` USING (`id_order`)
587: 					LEFT JOIN `order_driver_select` USING (`id_order`)
588: 					WHERE
589: 						`order`.`id_order_status` in (1,2,6) AND 
590: 						(
591: 							`order`.`datetime_start_plan` = 0 OR 
592: 							`order`.`datetime_start_plan` > now() - INTERVAL '86400' SECOND
593: 						)
594: 						AND 
595: 						(
596: 							(
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
```

### `archive_17012026_1259/taxi/config/system_bot.php:591`
```text
586: 					LEFT JOIN `order_driver` USING (`id_order`)
587: 					LEFT JOIN `order_driver_select` USING (`id_order`)
588: 					WHERE
589: 						`order`.`id_order_status` in (1,2,6) AND 
590: 						(
591: 							`order`.`datetime_start_plan` = 0 OR 
592: 							`order`.`datetime_start_plan` > now() - INTERVAL '86400' SECOND
593: 						)
594: 						AND 
595: 						(
596: 							(
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
598: 								`order_driver`.`id_order_driver_status` in (3,4,5)
599: 							)
```

### `archive_17012026_1259/taxi/config/system_bot.php:592`
```text
587: 					LEFT JOIN `order_driver_select` USING (`id_order`)
588: 					WHERE
589: 						`order`.`id_order_status` in (1,2,6) AND 
590: 						(
591: 							`order`.`datetime_start_plan` = 0 OR 
592: 							`order`.`datetime_start_plan` > now() - INTERVAL '86400' SECOND
593: 						)
594: 						AND 
595: 						(
596: 							(
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
598: 								`order_driver`.`id_order_driver_status` in (3,4,5)
599: 							)
600: 							OR
```

### `archive_17012026_1259/taxi/config/system_bot.php:597`
```text
592: 							`order`.`datetime_start_plan` > now() - INTERVAL '86400' SECOND
593: 						)
594: 						AND 
595: 						(
596: 							(
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
598: 								`order_driver`.`id_order_driver_status` in (3,4,5)
599: 							)
600: 							OR
601: 							(
602: 								`order_driver_select`.`id_user` = `users`.`id_user` AND 
603: 								`order_driver_select`.`cancel` = '0'
604: 							)
605: 						)
```

### `archive_17012026_1259/taxi/config/system_bot.php:598`
```text
593: 						)
594: 						AND 
595: 						(
596: 							(
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
598: 								`order_driver`.`id_order_driver_status` in (3,4,5)
599: 							)
600: 							OR
601: 							(
602: 								`order_driver_select`.`id_user` = `users`.`id_user` AND 
603: 								`order_driver_select`.`cancel` = '0'
604: 							)
605: 						)
606: 					LIMIT 1
```

### `archive_17012026_1259/taxi/config/system_bot.php:602`
```text
597: 								`order_driver`.`id_user` = `users`.`id_user` AND 
598: 								`order_driver`.`id_order_driver_status` in (3,4,5)
599: 							)
600: 							OR
601: 							(
602: 								`order_driver_select`.`id_user` = `users`.`id_user` AND 
603: 								`order_driver_select`.`cancel` = '0'
604: 							)
605: 						)
606: 					LIMIT 1
607: 				) IS NULL " . $sql_where_car_users . "
608: 			GROUP BY 
609: 				`users`.`id_user`
610: 			";
```

### `archive_17012026_1259/taxi/config/system_bot.php:951`
```text
946: 	if (!empty($drivers))
947: 	{
948: 		$s = array();
949: 		foreach($drivers as $id_order=>$user_list)
950: 		{
951: 			foreach($user_list as $id_user)
952: 			{
953: 				$s[] = "('" . $id_order . "', '" . $id_user . "', '" . $now_datetime . "')";
954: 			}
955: 		}
956: 		$s = "INSERT INTO `order_driver_select` (`id_order`,`id_user`,`create_datetime`) VALUES " . implode(",", $s);
957: 		$q = query($s);
958: 		if ($q === false) file_put_contents($log, "$now_datetime : array drivers - database insert failed\n", FILE_APPEND);
959: 	}
```

### `archive_17012026_1259/taxi/config/system_bot.php:953`
```text
948: 		$s = array();
949: 		foreach($drivers as $id_order=>$user_list)
950: 		{
951: 			foreach($user_list as $id_user)
952: 			{
953: 				$s[] = "('" . $id_order . "', '" . $id_user . "', '" . $now_datetime . "')";
954: 			}
955: 		}
956: 		$s = "INSERT INTO `order_driver_select` (`id_order`,`id_user`,`create_datetime`) VALUES " . implode(",", $s);
957: 		$q = query($s);
958: 		if ($q === false) file_put_contents($log, "$now_datetime : array drivers - database insert failed\n", FILE_APPEND);
959: 	}
960: });
```

### `archive_17012026_1259/taxi/config/system_bot.php:956`
```text
951: 			foreach($user_list as $id_user)
952: 			{
953: 				$s[] = "('" . $id_order . "', '" . $id_user . "', '" . $now_datetime . "')";
954: 			}
955: 		}
956: 		$s = "INSERT INTO `order_driver_select` (`id_order`,`id_user`,`create_datetime`) VALUES " . implode(",", $s);
957: 		$q = query($s);
958: 		if ($q === false) file_put_contents($log, "$now_datetime : array drivers - database insert failed\n", FILE_APPEND);
959: 	}
960: });
```

### `archive_17012026_1259/taxi/controllers/c_api.php:166`
```text
161: 								{
162: 									$out = $API->editWaitingTime(trim($_GET['par'][3]),isset($_POST['previous'])?trim($_POST['previous']):NULL,isset($_POST['additional'])?trim($_POST['additional']):NULL);
163: 								}
164: 								if ($_POST['action'] == 'set_performer')
165: 								{
166: 									$out = $API->setDriver(isset($_POST['data'])?$_POST['data']:'',trim($_GET['par'][3]),!empty($_POST['u_id'])?trim($_POST['u_id']):'',empty($_POST['performer'])?0:1,isset($_POST['b_driver_code'])?trim($_POST['b_driver_code']):NULL,isset($_POST['t_id'])?trim($_POST['t_id']):'');
167: 								}
168: 								elseif ($_POST['action'] == 'set_arrive_state')
169: 								{
170: 									$out = $API->setCarIsArrived(trim($_GET['par'][3]));
171: 								}
172: 								elseif ($_POST['action'] == 'set_start_state')
173: 								{
174: 									$out = $API->startOrder(trim($_GET['par'][3]));
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:158`
```text
153: 								{
154: 									$out = $API->editWaitingTime(trim($_GET['par'][3]),isset($_POST['previous'])?trim($_POST['previous']):NULL,isset($_POST['additional'])?trim($_POST['additional']):NULL);
155: 								}
156: 								if ($_POST['action'] == 'set_performer')
157: 								{
158: 									$out = $API->setDriver(isset($_POST['data'])?$_POST['data']:'',trim($_GET['par'][3]),!empty($_POST['u_id'])?trim($_POST['u_id']):'',empty($_POST['performer'])?0:1,isset($_POST['b_driver_code'])?trim($_POST['b_driver_code']):NULL,isset($_POST['t_id'])?trim($_POST['t_id']):'');
159: 								}
160: 								elseif ($_POST['action'] == 'set_arrive_state')
161: 								{
162: 									$out = $API->setCarIsArrived(trim($_GET['par'][3]));
163: 								}
164: 								elseif ($_POST['action'] == 'set_start_state')
165: 								{
166: 									$out = $API->startOrder(trim($_GET['par'][3]));
```

### `archive_17012026_1259/taxi/models/api.php:4515`
```text
4510: 			if (!empty($field_flag['b_offers']))
4511: 			{
4512: 				$sql_order .= "(SELECT
4513: 						GROUP_CONCAT(
4514: 							CONCAT_WS('|',
4515: 								`id_user`,
4516: 								`create_datetime`
4517: 							)
4518: 						SEPARATOR ';') 
4519: 					 FROM `order_driver_select`
4520: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4521: 					 ORDER BY `order_driver_select`.`id_user`
4522: 					) as b_offers,";
4523: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4520`
```text
4515: 								`id_user`,
4516: 								`create_datetime`
4517: 							)
4518: 						SEPARATOR ';') 
4519: 					 FROM `order_driver_select`
4520: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4521: 					 ORDER BY `order_driver_select`.`id_user`
4522: 					) as b_offers,";
4523: 			}
4524: 			if (!empty($field_flag['b_offer']))
4525: 			{
4526: 				$sql_order .= "(SELECT
4527: 						COUNT(`id_user`)
4528: 					 FROM `order_driver_select`
```

### `archive_17012026_1259/taxi/models/api.php:4521`
```text
4516: 								`create_datetime`
4517: 							)
4518: 						SEPARATOR ';') 
4519: 					 FROM `order_driver_select`
4520: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4521: 					 ORDER BY `order_driver_select`.`id_user`
4522: 					) as b_offers,";
4523: 			}
4524: 			if (!empty($field_flag['b_offer']))
4525: 			{
4526: 				$sql_order .= "(SELECT
4527: 						COUNT(`id_user`)
4528: 					 FROM `order_driver_select`
4529: 					 WHERE 
```

### `archive_17012026_1259/taxi/models/api.php:4527`
```text
4522: 					) as b_offers,";
4523: 			}
4524: 			if (!empty($field_flag['b_offer']))
4525: 			{
4526: 				$sql_order .= "(SELECT
4527: 						COUNT(`id_user`)
4528: 					 FROM `order_driver_select`
4529: 					 WHERE 
4530: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4531: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4532: 					 LIMIT 1
4533: 					) as b_offer,";
4534: 			}
4535: 			if (DEFAULT_PROFILE == 'stadium')
```

### `archive_17012026_1259/taxi/models/api.php:4530`
```text
4525: 			{
4526: 				$sql_order .= "(SELECT
4527: 						COUNT(`id_user`)
4528: 					 FROM `order_driver_select`
4529: 					 WHERE 
4530: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4531: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4532: 					 LIMIT 1
4533: 					) as b_offer,";
4534: 			}
4535: 			if (DEFAULT_PROFILE == 'stadium')
4536: 			{
4537: 				$sql_order .= "(SELECT
4538: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
```

### `archive_17012026_1259/taxi/models/api.php:4531`
```text
4526: 				$sql_order .= "(SELECT
4527: 						COUNT(`id_user`)
4528: 					 FROM `order_driver_select`
4529: 					 WHERE 
4530: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4531: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4532: 					 LIMIT 1
4533: 					) as b_offer,";
4534: 			}
4535: 			if (DEFAULT_PROFILE == 'stadium')
4536: 			{
4537: 				$sql_order .= "(SELECT
4538: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
4539: 						 FROM
```

### `archive_17012026_1259/taxi/models/api.php:4575`
```text
4570: 						`order`.`max_waiting_datetime` as b_max_waiting,
4571: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
4572: 						`order`.`code` as b_driver_code,
4573: 						`order`.`options` as b_options,
4574: 						`order`.`contact` as b_contact,
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
```

### `archive_17012026_1259/taxi/models/api.php:4576`
```text
4571: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
4572: 						`order`.`code` as b_driver_code,
4573: 						`order`.`options` as b_options,
4574: 						`order`.`contact` as b_contact,
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
```

### `archive_17012026_1259/taxi/models/api.php:4577`
```text
4572: 						`order`.`code` as b_driver_code,
4573: 						`order`.`options` as b_options,
4574: 						`order`.`contact` as b_contact,
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
```

### `archive_17012026_1259/taxi/models/api.php:4578`
```text
4573: 						`order`.`options` as b_options,
4574: 						`order`.`contact` as b_contact,
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4579`
```text
4574: 						`order`.`contact` as b_contact,
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4580`
```text
4575: 						`order`.`id_order_location` as b_location_class,
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4581`
```text
4576: 						`order`.`distance_estimate` as b_distance_estimate,
4577: 						`order`.`price_estimate` as b_price_estimate,
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:4583`
```text
4578: 						`order`.`currency` as b_currency,
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:4584`
```text
4579: 						`order`.`night_time` as b_night,
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:4585`
```text
4580: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4581: 							GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` = 2,NULL,
4582: 								CONCAT_WS(0x00,
4583: 									`order_driver`.`id_user`,
4584: 									`order_driver`.`id_car`,
4585: 									`order_driver`.`id_order_driver_status`,
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
```

### `archive_17012026_1259/taxi/models/api.php:4605`
```text
4600: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
4601: 						) as b_comments,
4602: 						(SELECT
4603: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
4604: 						 FROM `order_service`
4605: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4606: 						) as b_services
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
```

### `archive_17012026_1259/taxi/models/api.php:4607`
```text
4602: 						(SELECT
4603: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
4604: 						 FROM `order_service`
4605: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4606: 						) as b_services
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
```

### `archive_17012026_1259/taxi/models/api.php:4608`
```text
4603: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
4604: 						 FROM `order_service`
4605: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4606: 						) as b_services
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4609`
```text
4604: 						 FROM `order_service`
4605: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4606: 						) as b_services
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
4617: 			else
```

### `archive_17012026_1259/taxi/models/api.php:4611`
```text
4606: 						) as b_services
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
4617: 			else
4618: 			{
4619: 				$s = "SELECT
```

### `archive_17012026_1259/taxi/models/api.php:4612`
```text
4607: 					FROM `order` 
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
4617: 			else
4618: 			{
4619: 				$s = "SELECT
4620: 						`order`.`id_order` as b_id,
```

### `archive_17012026_1259/taxi/models/api.php:4613`
```text
4608: 					LEFT JOIN `order_driver` USING (`id_order`)
4609: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4610: 					WHERE	
4611: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (1,2,5,6)
4612: 					GROUP BY `order`.`id_order`
4613: 					ORDER BY `order`.`last_edit_datetime` DESC
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
4617: 			else
4618: 			{
4619: 				$s = "SELECT
4620: 						`order`.`id_order` as b_id,
4621: 						" . $sql_order . "
```

### `archive_17012026_1259/taxi/models/api.php:4646`
```text
4641: 						`order`.`approve_datetime` as b_approved,
4642: 						`order`.`max_waiting_datetime` as b_max_waiting,
4643: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
4644: 						`order`.`options` as b_options,
4645: 						`order`.`contact` as b_contact,
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
```

### `archive_17012026_1259/taxi/models/api.php:4647`
```text
4642: 						`order`.`max_waiting_datetime` as b_max_waiting,
4643: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
4644: 						`order`.`options` as b_options,
4645: 						`order`.`contact` as b_contact,
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
```

### `archive_17012026_1259/taxi/models/api.php:4648`
```text
4643: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
4644: 						`order`.`options` as b_options,
4645: 						`order`.`contact` as b_contact,
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
```

### `archive_17012026_1259/taxi/models/api.php:4649`
```text
4644: 						`order`.`options` as b_options,
4645: 						`order`.`contact` as b_contact,
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4650`
```text
4645: 						`order`.`contact` as b_contact,
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4651`
```text
4646: 						`order`.`id_order_location` as b_location_class,
4647: 						`order`.`distance_estimate` as b_distance_estimate,
4648: 						`order`.`price_estimate` as b_price_estimate,
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
```

### `archive_17012026_1259/taxi/models/api.php:4654`
```text
4649: 						`order`.`currency` as b_currency,
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:4655`
```text
4650: 						`order`.`night_time` as b_night,
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:4656`
```text
4651: 						IF(`order_driver`.`id_order` IS NULL,NULL,
4652: 							GROUP_CONCAT(
4653: 								CONCAT_WS(0x00,
4654: 									`order_driver`.`id_user`,
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
```

### `archive_17012026_1259/taxi/models/api.php:4660`
```text
4655: 									`order_driver`.`id_car`,
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
```

### `archive_17012026_1259/taxi/models/api.php:4661`
```text
4656: 									`order_driver`.`id_order_driver_status`,
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
4669: 						(SELECT
```

### `archive_17012026_1259/taxi/models/api.php:4662`
```text
4657: 									IFNULL(`users_location`.`lat`,''),
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
4669: 						(SELECT
4670: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
```

### `archive_17012026_1259/taxi/models/api.php:4663`
```text
4658: 									IFNULL(`users_location`.`lng`,''),
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
4669: 						(SELECT
4670: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4671: 						 FROM `order_comment_items`
```

### `archive_17012026_1259/taxi/models/api.php:4664`
```text
4659: 									IFNULL(`users_location`.`datetime`,''),
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
4669: 						(SELECT
4670: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4671: 						 FROM `order_comment_items`
4672: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
```

### `archive_17012026_1259/taxi/models/api.php:4665`
```text
4660: 									`order_driver`.`candidacy_datetime`,
4661: 									`order_driver`.`appoint_datetime`,
4662: 									`order_driver`.`arrive_datetime`,
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
4666: 								)
4667: 							SEPARATOR 0x01) 
4668: 						) as drivers,
4669: 						(SELECT
4670: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4671: 						 FROM `order_comment_items`
4672: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
4673: 						) as b_comments,
```

### `archive_17012026_1259/taxi/models/api.php:4677`
```text
4672: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
4673: 						) as b_comments,
4674: 						(SELECT
4675: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
4676: 						 FROM `order_service`
4677: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4678: 						) as b_services,
4679: 						od.`id_order_driver_status` as c_state
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
```

### `archive_17012026_1259/taxi/models/api.php:4680`
```text
4675: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
4676: 						 FROM `order_service`
4677: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4678: 						) as b_services,
4679: 						od.`id_order_driver_status` as c_state
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
```

### `archive_17012026_1259/taxi/models/api.php:4681`
```text
4676: 						 FROM `order_service`
4677: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4678: 						) as b_services,
4679: 						od.`id_order_driver_status` as c_state
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
```

### `archive_17012026_1259/taxi/models/api.php:4682`
```text
4677: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
4678: 						) as b_services,
4679: 						od.`id_order_driver_status` as c_state
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
```

### `archive_17012026_1259/taxi/models/api.php:4683`
```text
4678: 						) as b_services,
4679: 						od.`id_order_driver_status` as c_state
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
4691: 							 FROM `order_driver_attempt`
```

### `archive_17012026_1259/taxi/models/api.php:4685`
```text
4680: 					FROM `order` 
4681: 					LEFT JOIN `order_driver` as od USING (`id_order`)
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
4691: 							 FROM `order_driver_attempt`
4692: 							 WHERE 
4693: 								`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:4687`
```text
4682: 					LEFT JOIN `order_driver` USING (`id_order`)
4683: 					LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
4684: 					WHERE
4685: 						((`order`.`id_order_status` in (1,6) AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4686: 						 ((od.`id_order_driver_status` = '1' AND 
4687: 						 `order`.`max_waiting_datetime` > now() AND
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
4691: 							 FROM `order_driver_attempt`
4692: 							 WHERE 
4693: 								`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
4694: 								`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
```

### `archive_17012026_1259/taxi/models/api.php:4693`
```text
4688: 						 (SELECT
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
4691: 							 FROM `order_driver_attempt`
4692: 							 WHERE 
4693: 								`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
4694: 								`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
```

### `archive_17012026_1259/taxi/models/api.php:4694`
```text
4689: 								COUNT(`order_driver_attempt`.`id_order`)
4690: 
4691: 							 FROM `order_driver_attempt`
4692: 							 WHERE 
4693: 								`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
4694: 								`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
```

### `archive_17012026_1259/taxi/models/api.php:4697`
```text
4692: 							 WHERE 
4693: 								`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
4694: 								`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
4703: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4704: 					";
4705: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4699`
```text
4694: 								`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
4703: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4704: 					";
4705: 			}
4706: 
4707: 			$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:4700`
```text
4695: 						 ) < " . $this->constant['driver_code_attempt_count_max'] . "
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
4703: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4704: 					";
4705: 			}
4706: 
4707: 			$q = query($s);
4708: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

### `archive_17012026_1259/taxi/models/api.php:4701`
```text
4696: 						 ) OR od.`id_order_driver_status` NOT in (1,2))) OR
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
4703: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4704: 					";
4705: 			}
4706: 
4707: 			$q = query($s);
4708: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
4709: 			
```

### `archive_17012026_1259/taxi/models/api.php:4702`
```text
4697: 						(`order`.`id_order_status` = '2' AND od.`id_user` = '" . $_SESSION[UID] . "' AND
4698: 						 od.`id_order_driver_status` NOT in (1,2))) AND
4699: 						(`order_driver`.`id_user` = '" . $_SESSION[UID] . "' OR 
4700: 						`order_driver`.`id_order_driver_status` NOT in (1,2))
4701: 					GROUP BY `order`.`id_order`
4702: 					ORDER BY `order`.`last_edit_datetime` DESC
4703: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4704: 					";
4705: 			}
4706: 
4707: 			$q = query($s);
4708: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
4709: 			
4710: 			$out = array('booking' => array(),'user' => array());
```

### `archive_17012026_1259/taxi/models/api.php:4982`
```text
4977: 			if (!empty($field_flag['b_offers']))
4978: 			{
4979: 				$sql_order .= "(SELECT
4980: 						GROUP_CONCAT(
4981: 							CONCAT_WS('|',
4982: 								`id_user`,
4983: 								`create_datetime`
4984: 							)
4985: 						SEPARATOR ';') 
4986: 					 FROM `order_driver_select`
4987: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4988: 					 ORDER BY `order_driver_select`.`id_user`
4989: 					) as b_offers,";
4990: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4987`
```text
4982: 								`id_user`,
4983: 								`create_datetime`
4984: 							)
4985: 						SEPARATOR ';') 
4986: 					 FROM `order_driver_select`
4987: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4988: 					 ORDER BY `order_driver_select`.`id_user`
4989: 					) as b_offers,";
4990: 			}
4991: 			if (!empty($field_flag['b_offer']))
4992: 			{
4993: 				$sql_order .= "(SELECT
4994: 						COUNT(`id_user`)
4995: 					 FROM `order_driver_select`
```

### `archive_17012026_1259/taxi/models/api.php:4988`
```text
4983: 								`create_datetime`
4984: 							)
4985: 						SEPARATOR ';') 
4986: 					 FROM `order_driver_select`
4987: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
4988: 					 ORDER BY `order_driver_select`.`id_user`
4989: 					) as b_offers,";
4990: 			}
4991: 			if (!empty($field_flag['b_offer']))
4992: 			{
4993: 				$sql_order .= "(SELECT
4994: 						COUNT(`id_user`)
4995: 					 FROM `order_driver_select`
4996: 					 WHERE 
```

### `archive_17012026_1259/taxi/models/api.php:4994`
```text
4989: 					) as b_offers,";
4990: 			}
4991: 			if (!empty($field_flag['b_offer']))
4992: 			{
4993: 				$sql_order .= "(SELECT
4994: 						COUNT(`id_user`)
4995: 					 FROM `order_driver_select`
4996: 					 WHERE 
4997: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4998: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4999: 					 LIMIT 1
5000: 					) as b_offer,";
5001: 			}
5002: 
```

### `archive_17012026_1259/taxi/models/api.php:4997`
```text
4992: 			{
4993: 				$sql_order .= "(SELECT
4994: 						COUNT(`id_user`)
4995: 					 FROM `order_driver_select`
4996: 					 WHERE 
4997: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4998: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4999: 					 LIMIT 1
5000: 					) as b_offer,";
5001: 			}
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
```

### `archive_17012026_1259/taxi/models/api.php:4998`
```text
4993: 				$sql_order .= "(SELECT
4994: 						COUNT(`id_user`)
4995: 					 FROM `order_driver_select`
4996: 					 WHERE 
4997: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
4998: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
4999: 					 LIMIT 1
5000: 					) as b_offer,";
5001: 			}
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
5006: 					`order`.`id_payment_card` as b_payment_card,
```

### `archive_17012026_1259/taxi/models/api.php:5005`
```text
5000: 					) as b_offer,";
5001: 			}
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
5006: 					`order`.`id_payment_card` as b_payment_card,
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
```

### `archive_17012026_1259/taxi/models/api.php:5006`
```text
5001: 			}
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
5006: 					`order`.`id_payment_card` as b_payment_card,
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
```

### `archive_17012026_1259/taxi/models/api.php:5007`
```text
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
5006: 					`order`.`id_payment_card` as b_payment_card,
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
```

### `archive_17012026_1259/taxi/models/api.php:5008`
```text
5003: 			if ($this->id_role == 4)
5004: 			{
5005: 				$sql_payment .= "					`order`.`id_payment_method` as b_payment_way,
5006: 					`order`.`id_payment_card` as b_payment_card,
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
```

### `archive_17012026_1259/taxi/models/api.php:5011`
```text
5006: 					`order`.`id_payment_card` as b_payment_card,
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
```

### `archive_17012026_1259/taxi/models/api.php:5012`
```text
5007: 					`order`.`code` as b_driver_code,";
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
```

### `archive_17012026_1259/taxi/models/api.php:5013`
```text
5008: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
```

### `archive_17012026_1259/taxi/models/api.php:5014`
```text
5009: 						GROUP_CONCAT(
5010: 							CONCAT_WS('|',
5011: 								`order_driver`.`id_user`,
5012: 								`order_driver`.`id_car`,
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
```

### `archive_17012026_1259/taxi/models/api.php:5018`
```text
5013: 								`order_driver`.`id_order_driver_status`,
5014: 								`order_driver`.`arrive_datetime`
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
```

### `archive_17012026_1259/taxi/models/api.php:5022`
```text
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
```

### `archive_17012026_1259/taxi/models/api.php:5023`
```text
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
5031: 					) as drivers,";
```

### `archive_17012026_1259/taxi/models/api.php:5025`
```text
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
5031: 					) as drivers,";
5032: 				$sql_left_join .= "
5033: 					LEFT JOIN (
```

### `archive_17012026_1259/taxi/models/api.php:5026`
```text
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
5031: 					) as drivers,";
5032: 				$sql_left_join .= "
5033: 					LEFT JOIN (
5034: 							SELECT
```

### `archive_17012026_1259/taxi/models/api.php:5027`
```text
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
5031: 					) as drivers,";
5032: 				$sql_left_join .= "
5033: 					LEFT JOIN (
5034: 							SELECT
5035: 								`id_order`,
```

### `archive_17012026_1259/taxi/models/api.php:5028`
```text
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
5024: 							CONCAT_WS('|',
5025: 								`order_driver`.`id_user`,
5026: 								`order_driver`.`id_car`,
5027: 								`order_driver`.`id_order_driver_status`,
5028: 								`order_driver`.`arrive_datetime`
5029: 							)
5030: 						) SEPARATOR ';')
5031: 					) as drivers,";
5032: 				$sql_left_join .= "
5033: 					LEFT JOIN (
5034: 							SELECT
5035: 								`id_order`,
5036: 								`id_user`	
```

### `archive_17012026_1259/taxi/models/api.php:5036`
```text
5031: 					) as drivers,";
5032: 				$sql_left_join .= "
5033: 					LEFT JOIN (
5034: 							SELECT
5035: 								`id_order`,
5036: 								`id_user`	
5037: 							FROM
5038: 								`order_driver`
5039: 							WHERE
5040: 								`id_user` = '" . $_SESSION[UID] . "'
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
```

### `archive_17012026_1259/taxi/models/api.php:5038`
```text
5033: 					LEFT JOIN (
5034: 							SELECT
5035: 								`id_order`,
5036: 								`id_user`	
5037: 							FROM
5038: 								`order_driver`
5039: 							WHERE
5040: 								`id_user` = '" . $_SESSION[UID] . "'
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:5040`
```text
5035: 								`id_order`,
5036: 								`id_user`	
5037: 							FROM
5038: 								`order_driver`
5039: 							WHERE
5040: 								`id_user` = '" . $_SESSION[UID] . "'
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5048: 					) = 0))";
```

### `archive_17012026_1259/taxi/models/api.php:5042`
```text
5037: 							FROM
5038: 								`order_driver`
5039: 							WHERE
5040: 								`id_user` = '" . $_SESSION[UID] . "'
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5048: 					) = 0))";
5049: 				$sql_where .= " AND od.`id_user` IS NULL AND
5050: 					(SELECT
```

### `archive_17012026_1259/taxi/models/api.php:5046`
```text
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5048: 					) = 0))";
5049: 				$sql_where .= " AND od.`id_user` IS NULL AND
5050: 					(SELECT
5051: 							COUNT(`order_driver_attempt`.`id_order`)
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:5047`
```text
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5048: 					) = 0))";
5049: 				$sql_where .= " AND od.`id_user` IS NULL AND
5050: 					(SELECT
5051: 							COUNT(`order_driver_attempt`.`id_order`)
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
```

### `archive_17012026_1259/taxi/models/api.php:5049`
```text
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5048: 					) = 0))";
5049: 				$sql_where .= " AND od.`id_user` IS NULL AND
5050: 					(SELECT
5051: 							COUNT(`order_driver_attempt`.`id_order`)
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
5056: 					) < " . $this->constant['driver_code_attempt_count_max'] . " AND
5057: 					`order`.`client` != '" . $_SESSION[UID] . "'";
```

### `archive_17012026_1259/taxi/models/api.php:5054`
```text
5049: 				$sql_where .= " AND od.`id_user` IS NULL AND
5050: 					(SELECT
5051: 							COUNT(`order_driver_attempt`.`id_order`)
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
5056: 					) < " . $this->constant['driver_code_attempt_count_max'] . " AND
5057: 					`order`.`client` != '" . $_SESSION[UID] . "'";
5058: 			}
5059: 			$sql_distance = "";
5060: 			$sql_group = "GROUP BY `order`.`id_order`";
5061: 			$sql_having = "";
5062: 			$sql_order_by = "ORDER BY `order`.`last_edit_datetime` DESC";
```

### `archive_17012026_1259/taxi/models/api.php:5055`
```text
5050: 					(SELECT
5051: 							COUNT(`order_driver_attempt`.`id_order`)
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
5056: 					) < " . $this->constant['driver_code_attempt_count_max'] . " AND
5057: 					`order`.`client` != '" . $_SESSION[UID] . "'";
5058: 			}
5059: 			$sql_distance = "";
5060: 			$sql_group = "GROUP BY `order`.`id_order`";
5061: 			$sql_having = "";
5062: 			$sql_order_by = "ORDER BY `order`.`last_edit_datetime` DESC";
5063: 			$limit_offset = $this->limit_offset;
```

### `archive_17012026_1259/taxi/models/api.php:5057`
```text
5052: 						FROM `order_driver_attempt`
5053: 						WHERE 
5054: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
5056: 					) < " . $this->constant['driver_code_attempt_count_max'] . " AND
5057: 					`order`.`client` != '" . $_SESSION[UID] . "'";
5058: 			}
5059: 			$sql_distance = "";
5060: 			$sql_group = "GROUP BY `order`.`id_order`";
5061: 			$sql_having = "";
5062: 			$sql_order_by = "ORDER BY `order`.`last_edit_datetime` DESC";
5063: 			$limit_offset = $this->limit_offset;
5064: 			$limit_row_count = $this->limit_row_count;
5065: 			$sql_limit = "LIMIT " . $limit_offset . ", " . $limit_row_count;
```

### `archive_17012026_1259/taxi/models/api.php:5060`
```text
5055: 							`order_driver_attempt`.`id_user` = '" . $_SESSION[UID] . "'
5056: 					) < " . $this->constant['driver_code_attempt_count_max'] . " AND
5057: 					`order`.`client` != '" . $_SESSION[UID] . "'";
5058: 			}
5059: 			$sql_distance = "";
5060: 			$sql_group = "GROUP BY `order`.`id_order`";
5061: 			$sql_having = "";
5062: 			$sql_order_by = "ORDER BY `order`.`last_edit_datetime` DESC";
5063: 			$limit_offset = $this->limit_offset;
5064: 			$limit_row_count = $this->limit_row_count;
5065: 			$sql_limit = "LIMIT " . $limit_offset . ", " . $limit_row_count;
5066: 			$union = false;
5067: 			if (isset($filter) && $this->id_role == 2)
5068: 			{
```

### `archive_17012026_1259/taxi/models/api.php:5642`
```text
5637: 			if (!empty($field_flag['b_offers']))
5638: 			{
5639: 				$sql_order .= "(SELECT
5640: 						GROUP_CONCAT(
5641: 							CONCAT_WS('|',
5642: 								`id_user`,
5643: 								`create_datetime`
5644: 							)
5645: 						SEPARATOR ';') 
5646: 					 FROM `order_driver_select`
5647: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
5648: 					 ORDER BY `order_driver_select`.`id_user`
5649: 					) as b_offers,";
5650: 			}
```

### `archive_17012026_1259/taxi/models/api.php:5647`
```text
5642: 								`id_user`,
5643: 								`create_datetime`
5644: 							)
5645: 						SEPARATOR ';') 
5646: 					 FROM `order_driver_select`
5647: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
5648: 					 ORDER BY `order_driver_select`.`id_user`
5649: 					) as b_offers,";
5650: 			}
5651: 			if (!empty($field_flag['b_offer']))
5652: 			{
5653: 				$sql_order .= "(SELECT
5654: 						COUNT(`id_user`)
5655: 					 FROM `order_driver_select`
```

### `archive_17012026_1259/taxi/models/api.php:5648`
```text
5643: 								`create_datetime`
5644: 							)
5645: 						SEPARATOR ';') 
5646: 					 FROM `order_driver_select`
5647: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
5648: 					 ORDER BY `order_driver_select`.`id_user`
5649: 					) as b_offers,";
5650: 			}
5651: 			if (!empty($field_flag['b_offer']))
5652: 			{
5653: 				$sql_order .= "(SELECT
5654: 						COUNT(`id_user`)
5655: 					 FROM `order_driver_select`
5656: 					 WHERE 
```

### `archive_17012026_1259/taxi/models/api.php:5654`
```text
5649: 					) as b_offers,";
5650: 			}
5651: 			if (!empty($field_flag['b_offer']))
5652: 			{
5653: 				$sql_order .= "(SELECT
5654: 						COUNT(`id_user`)
5655: 					 FROM `order_driver_select`
5656: 					 WHERE 
5657: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
5658: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5659: 					 LIMIT 1
5660: 					) as b_offer,";
5661: 			}
5662: 			if (DEFAULT_PROFILE == 'stadium')
```

### `archive_17012026_1259/taxi/models/api.php:5657`
```text
5652: 			{
5653: 				$sql_order .= "(SELECT
5654: 						COUNT(`id_user`)
5655: 					 FROM `order_driver_select`
5656: 					 WHERE 
5657: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
5658: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5659: 					 LIMIT 1
5660: 					) as b_offer,";
5661: 			}
5662: 			if (DEFAULT_PROFILE == 'stadium')
5663: 			{
5664: 				$sql_order .= "(SELECT
5665: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
```

### `archive_17012026_1259/taxi/models/api.php:5658`
```text
5653: 				$sql_order .= "(SELECT
5654: 						COUNT(`id_user`)
5655: 					 FROM `order_driver_select`
5656: 					 WHERE 
5657: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
5658: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
5659: 					 LIMIT 1
5660: 					) as b_offer,";
5661: 			}
5662: 			if (DEFAULT_PROFILE == 'stadium')
5663: 			{
5664: 				$sql_order .= "(SELECT
5665: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
5666: 						 FROM
```

### `archive_17012026_1259/taxi/models/api.php:5723`
```text
5718: 						`order`.`complete_datetime` as b_completed,
5719: 						`order`.`max_waiting_datetime` as b_max_waiting,
5720: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
5721: 						`order`.`code` as b_driver_code,
5722: 						`order`.`options` as b_options,
5723: 						`order`.`id_order_location` as b_location_class,
5724: 						`order`.`distance_estimate` as b_distance_estimate,
5725: 						`order`.`price_estimate` as b_price_estimate,
5726: 						`order`.`currency` as b_currency,
5727: 						`order`.`night_time` as b_night,
5728: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
```

### `archive_17012026_1259/taxi/models/api.php:5724`
```text
5719: 						`order`.`max_waiting_datetime` as b_max_waiting,
5720: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
5721: 						`order`.`code` as b_driver_code,
5722: 						`order`.`options` as b_options,
5723: 						`order`.`id_order_location` as b_location_class,
5724: 						`order`.`distance_estimate` as b_distance_estimate,
5725: 						`order`.`price_estimate` as b_price_estimate,
5726: 						`order`.`currency` as b_currency,
5727: 						`order`.`night_time` as b_night,
5728: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
```

## 4. Критерий закрытия bridge

Bridge получает `CONFIRMED` только при наличии наблюдаемой цепочки:

```text
assigned performer field
        ↓
same value / derived user ID
        ↓
u_id
        ↓
/location
```

Если frontend выбирает `u_id` из другого источника, это не тот bridge.

## 5. Текущий результат

Поисковые contexts подтверждают наличие implementation fragments, но совместное присутствие `order_driver`, `id_driver`, `u_id` и `/location` в одном source corpus не доказывает data-flow.

Поэтому до установления конкретного assignment → `u_id` data-flow Claim остаётся:

```text
Order
  → EXPOSES_ASSIGNED_PERFORMER_POSITION
  = UNKNOWN
```

## 6. Отдельные Claims

```text
Order → ASSIGNED_PERFORMER → User
CONFIRMED

User → HAS_CURRENT_POSITION → Position
CONFIRMED

Frontend → CALLS /location
CONFIRMED where concrete call exists
```

Композиционный Claim требует direct data-flow Evidence.

## 7. Gap classification

Если конкретный frontend call `/location` использует assigned driver ID — bridge `CONFIRMED`.

Если используется current user ID или другой independently sourced ID — bridge для данного path `REJECTED`.

Если source недоступен/делегируется в неиндексированный слой — `UNKNOWN / SOURCE_GAP`.

## 8. Gap Report

```text
G-24-01  order response → assigned driver ID        OPEN
G-24-02  assigned driver ID → frontend u_id          OPEN
G-24-03  frontend u_id → /location                   OPEN
G-24-04  /location result → map rendering            OPEN
```

## 9. Следующий шаг

Не делать новый общий поиск.

Открыть конкретный frontend function, содержащий `/location`, и восстановить его caller/state source до значения `u_id`. Одновременно открыть конкретный backend response, который предоставляет assigned driver ID.

Цель:

```text
ONE frontend call
      +
ONE backend response
      ↓
ONE explicit data-flow bridge
```

Только после этого добавлять relation в Semantic Graph.