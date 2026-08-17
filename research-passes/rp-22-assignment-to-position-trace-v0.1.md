# Backend Semantic Graph — Research Pass 22
# Assignment → User → Position Trace v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-21 `setDriver` Full Authorization / Preconditions Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Research Question

> После успешного `setDriver` как в Core Backend представляется назначенный исполнитель, где хранится его текущая Position и существует ли доказанная цепочка до frontend-visible position?

Цель — соединить две ранее независимые ветви:

```text
Order → Assigned Performer
Order → User → Current Position
```

и не добавлять relation, если между ними нет Evidence.

## 2. Assignment Evidence

Найдено assignment-related contexts: **130**.
### `archive_17012026_1259/taxi/models/api.php:4212`
```text
4209: 	
4210: 			if (!empty($data['b_only_offer']))
4211: 			{
4212: 				$add_status = "`id_order_status` = '6', `offer_datetime` = now()";
4213: 			}
4214: 			elseif (empty($this->constant['d_s_bot']) || $datetime_start_plan === '0000-00-00 00:00:00')
4215: 			{
4216: 				$add_status = "`id_order_status` = '1', `process_datetime` = now()";
4217: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4216`
```text
4213: 			}
4214: 			elseif (empty($this->constant['d_s_bot']) || $datetime_start_plan === '0000-00-00 00:00:00')
4215: 			{
4216: 				$add_status = "`id_order_status` = '1', `process_datetime` = now()";
4217: 			}
4218: 			else
4219: 			{
4220: 				$add_status = "`id_order_status` = '5', `pending_datetime` = now()";
4221: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4220`
```text
4217: 			}
4218: 			else
4219: 			{
4220: 				$add_status = "`id_order_status` = '5', `pending_datetime` = now()";
4221: 			}
4222: 
4223: 			$s = "INSERT INTO `order`
4224: 				SET 
4225: 					" . implode(",\n", $s) .",
```

### `archive_17012026_1259/taxi/models/api.php:4565`
```text
4562: 						`order`.`luggage_count` as b_luggage_count,
4563: 						`order`.`placard` as b_placard,
4564: 						`order`.`id_car_class` as b_car_class,
4565: 						`order`.`id_order_status` as b_state,
4566: 						`order`.`create_datetime` as b_created,
4567: 						`order`.`is_confirmed` as b_confirm_state,
4568: 						`order`.`car_count` as b_cars_count,
4569: 						`order`.`approve_datetime` as b_approved,
4570: 						`order`.`max_waiting_datetime` as b_max_waiting,
```

### `archive_17012026_1259/taxi/models/api.php:4586`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4587`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4588`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4590`
```text
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
4594: 								)
4595: 							) SEPARATOR 0x01)
```

### `archive_17012026_1259/taxi/models/api.php:4609`
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
```

### `archive_17012026_1259/taxi/models/api.php:4611`
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
```

### `archive_17012026_1259/taxi/models/api.php:4637`
```text
4634: 						`order`.`luggage_count` as b_luggage_count,
4635: 						`order`.`placard` as b_placard,
4636: 						`order`.`id_car_class` as b_car_class,
4637: 						`order`.`id_order_status` as b_state,
4638: 						`order`.`create_datetime` as b_created,
4639: 						`order`.`is_confirmed` as b_confirm_state,
4640: 						`order`.`car_count` as b_cars_count,
4641: 						`order`.`approve_datetime` as b_approved,
4642: 						`order`.`max_waiting_datetime` as b_max_waiting,
```

### `archive_17012026_1259/taxi/models/api.php:4657`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4658`
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
```

### `archive_17012026_1259/taxi/models/api.php:4659`
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
```

### `archive_17012026_1259/taxi/models/api.php:4661`
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
```

### `archive_17012026_1259/taxi/models/api.php:4683`
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
```

### `archive_17012026_1259/taxi/models/api.php:4685`
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
```

### `archive_17012026_1259/taxi/models/api.php:4697`
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
```

### `archive_17012026_1259/taxi/models/api.php:5018`
```text
5015: 							)
5016: 						SEPARATOR ';') 
5017: 					) as drivers,";
5018: 				$sql_where_order .= "`order`.`id_order_status`in (1,6)";
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
5022: 				$sql_drivers .= "IF(`order_driver`.`id_order` IS NULL,NULL,
5023: 						GROUP_CONCAT(IF(`order_driver`.`id_order_driver_status` in (1,2),NULL,
```

### `archive_17012026_1259/taxi/models/api.php:5042`
```text
5039: 							WHERE
5040: 								`id_user` = '" . $_SESSION[UID] . "'
5041: 						) od USING (`id_order`)";
5042: 				$sql_where_order .= "(`order`.`id_order_status` = '1' OR (`order`.`id_order_status` = '6' AND (SELECT
5043: 							`order_driver_select`.`cancel`
5044: 						FROM `order_driver_select`
5045: 						WHERE 
5046: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
5047: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
```

### `archive_17012026_1259/taxi/models/api.php:5172`
```text
5169: 					`order`.`placard` as b_placard,
5170: 					`order`.`id_car_class` as b_car_class,
5171: 					" . $sql_payment . "
5172: 					`order`.`id_order_status` as b_state,
5173: 					`order`.`create_datetime` as b_created,
5174: 					`order`.`is_confirmed` as b_confirm_state,
5175: 					`order`.`car_count` as b_cars_count,
5176: 					`order`.`approve_datetime` as b_approved,
5177: 					`order`.`max_waiting_datetime` as b_max_waiting,
```

### `archive_17012026_1259/taxi/models/api.php:5695`
```text
5692: 						`order`.`id_payment_method` as b_payment_way,
5693: 						`order`.`id_payment_card` as b_payment_card,
5694: 						`order`.`tips` as b_tips,
5695: 						`order`.`id_order_status` as b_state,
5696: 						`order`.`rating` as b_rating,
5697: 						`order`.`create_datetime` as b_created,
5698: 						`order`.`is_confirmed` as b_confirm_state,
5699: 						`order`.`confirm_limit_datetime` as b_confirmation_limit,
5700: 						`order`.`confirm_datetime` as b_confirmation_datetime,
```

### `archive_17012026_1259/taxi/models/api.php:5737`
```text
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
5736: 									`order_driver`.`candidacy_datetime`,
5737: 									`order_driver`.`appoint_datetime`,
5738: 									`order_driver`.`cancel_datetime`,
5739: 									`order_driver`.`arrive_datetime`,
5740: 									`order_driver`.`start_datetime`,
5741: 									`order_driver`.`complete_datetime`,
5742: 									IFNULL(`order_driver`.`tips`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5760`
```text
5757: 					FROM `order` 
5758: 					LEFT JOIN `order_driver` USING (`id_order`)			
5759: 					WHERE	
5760: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (3,4)
5761: 					GROUP BY `order`.`id_order`
5762: 					ORDER BY `order`.`last_edit_datetime` DESC
5763: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
5764: 					";
5765: 			}
```

### `archive_17012026_1259/taxi/models/api.php:5787`
```text
5784: 						`order`.`placard` as b_placard,
5785: 						`order`.`id_car_class` as b_car_class,
5786: 						`order`.`tips` as b_tips,
5787: 						`order`.`id_order_status` as b_state,
5788: 						`order`.`rating` as b_rating,
5789: 						`order`.`create_datetime` as b_created,
5790: 						`order`.`is_confirmed` as b_confirm_state,					
5791: 						`order`.`car_count` as b_cars_count,
5792: 						`order`.`cancel_reason` as b_cancel_reason,
```

### `archive_17012026_1259/taxi/models/api.php:5828`
```text
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5854`
```text
5851: 					LEFT JOIN `order_driver` as od USING (`id_order`)
5852: 					LEFT JOIN `order_driver` USING (`id_order`)			
5853: 					WHERE
5854: 						(`order`.`id_order_status` in (3,4) OR od.`id_order_driver_status` = '2') AND 
5855: 						od.`id_user` = '" . $_SESSION[UID] . "' 
5856: 					GROUP BY `order`.`id_order`
5857: 					ORDER BY `order`.`last_edit_datetime` DESC
5858: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
5859: 					";
```

### `archive_17012026_1259/taxi/models/api.php:6180`
```text
6177: 			);
6178: 		}
6179: 
6180: 		public function setDriver($data = '', $id_order = "", $id_user = "", $appointed_performer = 0, $code = NULL, $trips = "")
6181: 		{
6182: 			if (empty($_SESSION[UID])) {
6183: 				return $this->showError('404', 'error', 'unauthorized access');
6184: 			}
6185: 
```

### `archive_17012026_1259/taxi/models/api.php:6196`
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
```

### `archive_17012026_1259/taxi/models/api.php:6214`
```text
6211: 				
6212: 				$d = fetch_assoc($q);
6213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6214: 				if ($d['id_order_status'] != 1) return $this->showError('404', 'error', 'wrong booking state');
6215: 
6216: 				if ($d['client'] != $_SESSION[UID]) 
6217: 				{
6218: 					return $this->showError('404', 'error', $_SESSION[UID] . ' is not author');
6219: 				}
```

### `archive_17012026_1259/taxi/models/api.php:6246`
```text
6243: 
6244: 						) od
6245: 					SET 
6246: 						`order`.`id_order_status` = (SELECT @id := 
6247: 							IF(od.`current_car_count` < `order`.`car_count`-1,1,2)),
6248: 						`order`.`approve_datetime` = 
6249: 							IF(od.`current_car_count` < `order`.`car_count`-1,`order`.`approve_datetime`,now()),	
6250: 						`order`.`car_count` = (SELECT @c_count := `order`.`car_count`),	
6251: 						`order_driver`.`id_order_driver_status` = 
```

### `archive_17012026_1259/taxi/models/api.php:6253`
```text
6250: 						`order`.`car_count` = (SELECT @c_count := `order`.`car_count`),	
6251: 						`order_driver`.`id_order_driver_status` = 
6252: 							IF( od.`current_car_count` < `order`.`car_count`,3,1),
6253: 						`order_driver`.`appoint_datetime` = 
6254: 							IF( od.`current_car_count` < `order`.`car_count`,now(),
6255: 							`order_driver`.`appoint_datetime`)
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:6255`
```text
6252: 							IF( od.`current_car_count` < `order`.`car_count`,3,1),
6253: 						`order_driver`.`appoint_datetime` = 
6254: 							IF( od.`current_car_count` < `order`.`car_count`,now(),
6255: 							`order_driver`.`appoint_datetime`)
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
```

### `archive_17012026_1259/taxi/models/api.php:6257`
```text
6254: 							IF( od.`current_car_count` < `order`.`car_count`,now(),
6255: 							`order_driver`.`appoint_datetime`)
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
```

### `archive_17012026_1259/taxi/models/api.php:6386`
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
```

### `archive_17012026_1259/taxi/models/api.php:6409`
```text
6406: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
6407: 							`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
6408: 						) as attempt,
6409: 						IF(`order`.`id_order_status` = '6',(
6410: 							SELECT
6411: 								`order_driver_select`.`id_user`
6412: 							 FROM `order_driver_select`
6413: 							 WHERE 
6414: 								`order_driver_select`.`id_order` = `order`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:6582`
```text
6579: 
6580: 				if (!empty($code))
6581: 				{
6582: 					if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 6) 
6583: 					{
6584: 						return $this->showError('404', 'error', 'booking with wrong state');
6585: 					}
6586: 					if ($d['car_count'] - ($d['current_car_count'] - $d['appointed_performer_count']) <= 0)
6587: 					{
```

### `archive_17012026_1259/taxi/models/api.php:6596`
```text
6593: 				}
6594: 				else
6595: 				{
6596: 					if ($d['id_order_status'] != 1) 
6597: 					{
6598: 						if ($d['id_order_status'] == 6)
6599: 						{
6600: 							if (empty($d['selected'])) return $this->showError('404', 'error', 'not selected driver');
6601: 							$appointed_performer = 1;
```

### `archive_17012026_1259/taxi/models/api.php:6598`
```text
6595: 				{
6596: 					if ($d['id_order_status'] != 1) 
6597: 					{
6598: 						if ($d['id_order_status'] == 6)
6599: 						{
6600: 							if (empty($d['selected'])) return $this->showError('404', 'error', 'not selected driver');
6601: 							$appointed_performer = 1;
6602: 						}
6603: 						else
```

### `archive_17012026_1259/taxi/models/api.php:6620`
```text
6617: 							return $this->showError('404', 'error', 'booking not confirmed');
6618: 						}
6619: 						$id_order_driver_status = '3';
6620: 						$datetime_prefix = "appoint_datetime";
6621: 						$d['current_car_count']++;
6622: 						$d['appointed_performer_count']++;
6623: 					}
6624: 				}
6625: 
```

### `archive_17012026_1259/taxi/models/api.php:6679`
```text
6676: 					$out = array('current_cars_count' => $d['current_car_count'],'b_cars_count' => $d['car_count']);
6677: 				}
6678: 
6679: 				if ($d['id_order_status'] != 2 && $d['current_car_count'] >= $d['car_count'])
6680: 				{
6681: 					$s = "UPDATE `order`
6682: 						SET 
6683: 							`id_order_status` = '2',
6684: 							`approve_datetime` = now()
```

### `archive_17012026_1259/taxi/models/api.php:6683`
```text
6680: 				{
6681: 					$s = "UPDATE `order`
6682: 						SET 
6683: 							`id_order_status` = '2',
6684: 							`approve_datetime` = now()
6685: 						WHERE
6686: 							`order`.`id_order` = '" . $id_order . "' AND `id_order_status` in (1,6)
6687: 						";
6688: 
```

### `archive_17012026_1259/taxi/models/api.php:6686`
```text
6683: 							`id_order_status` = '2',
6684: 							`approve_datetime` = now()
6685: 						WHERE
6686: 							`order`.`id_order` = '" . $id_order . "' AND `id_order_status` in (1,6)
6687: 						";
6688: 
6689: 					query($s);
6690: 
6691: 					if (mysqli_affected_rows($link) === -1) 
```

### `archive_17012026_1259/taxi/models/api.php:6811`
```text
6808: 
6809: 			$s = "SELECT
6810: 					`order`.`id_order`,
6811: 					`order`.`id_order_status`,
6812: 					od.`id_user`,
6813: 					od.`id_order_driver_status`
6814: 				FROM `order` 
6815: 				LEFT JOIN (
6816: 						SELECT
```

### `archive_17012026_1259/taxi/models/api.php:6835`
```text
6832: 
6833: 			$d = fetch_assoc($q);
6834: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6835: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 
6836: 				&& $d['id_order_status'] != 6) 
6837: 			{
6838: 				return $this->showError('404', 'error', 'wrong booking state');
6839: 			}
6840: 			if (empty($d['id_user'])) 
```

### `archive_17012026_1259/taxi/models/api.php:6836`
```text
6833: 			$d = fetch_assoc($q);
6834: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6835: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 
6836: 				&& $d['id_order_status'] != 6) 
6837: 			{
6838: 				return $this->showError('404', 'error', 'wrong booking state');
6839: 			}
6840: 			if (empty($d['id_user'])) 
6841: 			{
```

### `archive_17012026_1259/taxi/models/api.php:6911`
```text
6908: 				$s = "SELECT
6909: 						`id_order`,
6910: 						`client`,
6911: 						`id_order_status`
6912: 					FROM `order` 		
6913: 					WHERE	
6914: 						`id_order` = '" . $id_order . "'
6915: 					LIMIT 1
6916: 					";
```

### `archive_17012026_1259/taxi/models/api.php:6923`
```text
6920: 
6921: 				$d = fetch_assoc($q);
6922: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6923: 				if ($d['id_order_status'] != 2) return $this->showError('404', 'error', 'wrong booking state');
6924: 				if ($d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not author');
6925: 
6926: 				$s = "UPDATE `order`
6927: 					SET 
6928: 						`id_order_status` = '4',
```

### `archive_17012026_1259/taxi/models/api.php:6928`
```text
6925: 
6926: 				$s = "UPDATE `order`
6927: 					SET 
6928: 						`id_order_status` = '4',
6929: 						`complete_datetime` = now(),
6930: 						`last_edit_datetime` = now(),
6931: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
6932: 					WHERE
6933: 						`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
```

### `archive_17012026_1259/taxi/models/api.php:6933`
```text
6930: 						`last_edit_datetime` = now(),
6931: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
6932: 					WHERE
6933: 						`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
6934: 					";
6935: 
6936: 				query($s);
6937: 		
6938: 				global $link;
```

### `archive_17012026_1259/taxi/models/api.php:6953`
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
```

### `archive_17012026_1259/taxi/models/api.php:6974`
```text
6971: 
6972: 				$d = fetch_assoc($q);
6973: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6974: 				if ($d['id_order_status'] != 2 && $d['id_order_status'] != 4)
6975: 				{
6976: 					return $this->showError('404', 'error', 'wrong booking state');
6977: 				}
6978: 				if (empty($d['u_id'])) 
6979: 				{
```

### `archive_17012026_1259/taxi/models/api.php:7019`
```text
7016: 						return $this->showError('404', 'error', 'driver modified data not found');
7017: 					}
7018: 				}
7019: 				if ($d['id_order_status'] == 2 && empty($d['incomplete_count']))
7020: 				{
7021: 					$s = "UPDATE `order`
7022: 						SET 
7023: 							`id_order_status` = '4',
7024: 							`complete_datetime` = now(),
```

### `archive_17012026_1259/taxi/models/api.php:7023`
```text
7020: 				{
7021: 					$s = "UPDATE `order`
7022: 						SET 
7023: 							`id_order_status` = '4',
7024: 							`complete_datetime` = now(),
7025: 							`last_edit_datetime` = now(),
7026: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7027: 						WHERE
7028: 							`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
```

### `archive_17012026_1259/taxi/models/api.php:7028`
```text
7025: 							`last_edit_datetime` = now(),
7026: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7027: 						WHERE
7028: 							`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
7029: 						";
7030: 
7031: 					query($s);
7032: 			
7033: 					if (mysqli_affected_rows($link) === -1) 
```

### `archive_17012026_1259/taxi/models/api.php:7087`
```text
7084: 				$s = "SELECT
7085: 						`id_order`,
7086: 						`client`,
7087: 						`id_order_status`,
7088: 						`pay_datetime`,
7089: 						`options`
7090: 					FROM `order` 		
7091: 					WHERE	
7092: 						`id_order` = '" . $id_order . "'
```

### `archive_17012026_1259/taxi/models/api.php:7101`
```text
7098: 
7099: 				$d = fetch_assoc($q);
7100: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7101: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5 
7102: 					&& $d['id_order_status'] != 6) 
7103: 				{
7104: 					return $this->showError('404', 'error', 'wrong booking state');
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
```

### `archive_17012026_1259/taxi/models/api.php:7102`
```text
7099: 				$d = fetch_assoc($q);
7100: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7101: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5 
7102: 					&& $d['id_order_status'] != 6) 
7103: 				{
7104: 					return $this->showError('404', 'error', 'wrong booking state');
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
7107: 				{
```

### `archive_17012026_1259/taxi/models/api.php:7160`
```text
7157: 
7158: 					$s = "UPDATE `order`
7159: 						SET 
7160: 							`id_order_status` = '3',
7161: 							`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7162: 							`cancel_datetime` = now(),
7163: 							`last_edit_datetime` = now(),
7164: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7165: 						WHERE
```

### `archive_17012026_1259/taxi/models/api.php:7166`
```text
7163: 							`last_edit_datetime` = now(),
7164: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7165: 						WHERE
7166: 							`id_order` = '" . $id_order . "' AND `id_order_status`in (1,2,5,6)
7167: 						";
7168: 
7169: 					query($s);
7170: 			
7171: 					global $link;
```

### `archive_17012026_1259/taxi/models/api.php:7189`
```text
7186: 			{
7187: 				$s = "SELECT
7188: 						`order`.`id_order`,
7189: 						`order`.`id_order_status`,
7190: 						od.`id_user`,
7191: 						od.`id_order_driver_status`
7192: 					FROM `order` 
7193: 					LEFT JOIN (
7194: 							SELECT
```

### `archive_17012026_1259/taxi/models/api.php:7214`
```text
7211: 
7212: 				$d = fetch_assoc($q);
7213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7214: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2
7215: 					&& $d['id_order_status'] != 6) 
7216: 				{
7217: 					return $this->showError('404', 'error', 'wrong booking state');
7218: 				}
7219: 				if (empty($d['id_user'])) 
```

### `archive_17012026_1259/taxi/models/api.php:7215`
```text
7212: 				$d = fetch_assoc($q);
7213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7214: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2
7215: 					&& $d['id_order_status'] != 6) 
7216: 				{
7217: 					return $this->showError('404', 'error', 'wrong booking state');
7218: 				}
7219: 				if (empty($d['id_user'])) 
7220: 				{
```

### `archive_17012026_1259/taxi/models/api.php:7282`
```text
7279: 					
7280: 					$s = "UPDATE `order`,`order_driver`
7281: 						SET 
7282: 							`order`.`id_order_status` = IF(
7283: 								(SELECT 
7284: 									current_car_count 
7285: 								 FROM (
7286: 										SELECT
7287: 											COUNT(`id_order`) as current_car_count
```

### `archive_17012026_1259/taxi/models/api.php:7295`
```text
7292: 											`id_order_driver_status` in (3,4,5,6)
7293: 									) od
7294: 								) < `order`.`car_count`+1,
7295: 								IF(`order`.`id_order_status` in (1,5,6),`order`.`id_order_status`,1),2),
7296: 							`order_driver`.`id_order_driver_status` = '2',
7297: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7298: 							`order_driver`.`cancel_datetime` = now(),
7299: 							`order_driver`.`not_deleted` = NULL
7300: 						WHERE
```

### `archive_17012026_1259/taxi/models/api.php:7301`
```text
7298: 							`order_driver`.`cancel_datetime` = now(),
7299: 							`order_driver`.`not_deleted` = NULL
7300: 						WHERE
7301: 							`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` in (1,2,5,6) AND
7302: 							`order`.`id_order` = `order_driver`.`id_order` AND 
7303: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7304: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
7305: 							`order_driver`.`id_order_driver_status` in (3,4)
7306: 						";
```

### `archive_17012026_1259/taxi/models/api.php:7355`
```text
7352: 						SET 
7353: 							`order`.`last_edit_datetime` = now(),
7354: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7355: 							`order`.`id_order_status` = '3',
7356: 							`order`.`cancel_datetime` = now(),
7357: 							`order_driver`.`id_order_driver_status` = '2',
7358: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7359: 							`order_driver`.`cancel_datetime` = now(),
7360: 							`order_driver`.`not_deleted` = NULL
```

### `archive_17012026_1259/taxi/models/api.php:7362`
```text
7359: 							`order_driver`.`cancel_datetime` = now(),
7360: 							`order_driver`.`not_deleted` = NULL
7361: 						WHERE
7362: 							`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` in (6,1,2) AND
7363: 
7364: 							`order`.`id_order` = `order_driver`.`id_order` AND 
7365: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7366: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
7367: 							`order_driver`.`id_order_driver_status` = '5'
```

### `archive_17012026_1259/taxi/models/api.php:7411`
```text
7408: 				$s = "SELECT
7409: 						`id_order`,
7410: 						`client`,
7411: 						`id_order_status`,
7412: 						`rating`
7413: 					FROM `order` 		
7414: 					WHERE	
7415: 						`id_order` = '" . $id_order . "'
7416: 					LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:7424`
```text
7421: 
7422: 				$d = fetch_assoc($q);
7423: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7424: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
7425: 				if ($d['client'] != $_SESSION[UID]) 
7426: 				{
7427: 					return $this->showError('404', 'error', 'user is not author');
7428: 				}
7429: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
```

### `archive_17012026_1259/taxi/models/api.php:7457`
```text
7454: 			{
7455: 				$s = "SELECT
7456: 						`order`.`id_order`,
7457: 						`order`.`id_order_status`,
7458: 						od.`id_user`,
7459: 						od.`id_order_driver_status`,
7460: 						od.`rating`
7461: 					FROM `order`
7462: 					LEFT JOIN (
```

### `archive_17012026_1259/taxi/models/api.php:7483`
```text
7480: 
7481: 				$d = fetch_assoc($q);
7482: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7483: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
7484: 				if (empty($d['id_user'])) 
7485: 				{
7486: 					return $this->showError('404', 'error', 'user is not performer');
7487: 				}
7488: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
```

### `archive_17012026_1259/taxi/models/api.php:7662`
```text
7659: 							WHERE
7660: 								`id_user` = '" . $_SESSION[UID] . "'
7661: 						) od USING (`id_order`)";
7662: 				$sql_where .= " AND ((`order`.`id_order_status` = '1' AND `order`.`max_waiting_datetime` > now() AND od.`id_user` IS NULL AND
7663: 					(SELECT
7664: 							COUNT(`order_driver_attempt`.`id_order`)
7665: 						FROM `order_driver_attempt`
7666: 						WHERE 
7667: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
```

### `archive_17012026_1259/taxi/models/api.php:7711`
```text
7708: 					`order`.`id_car_class` as b_car_class,
7709: 					" . $sql_order . "
7710: 					`order`.`tips` as b_tips,
7711: 					`order`.`id_order_status` as b_state,
7712: 					`order`.`rating` as b_rating,
7713: 					`order`.`create_datetime` as b_created,
7714: 					`order`.`is_confirmed` as b_confirm_state,
7715: 					`order`.`car_count` as b_cars_count,
7716: 					`order`.`cancel_reason` as b_cancel_reason,
```

### `archive_17012026_1259/taxi/models/api.php:7757`
```text
7754: 								`order_driver`.`id_user`,
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:7758`
```text
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7759`
```text
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7764`
```text
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
7765: 								`order_driver`.`cancel_datetime`,
7766: 								`order_driver`.`arrive_datetime`,
7767: 								`order_driver`.`start_datetime`,
7768: 								`order_driver`.`complete_datetime`,
7769: 								IFNULL(`order_driver`.`tips`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:8194`
```text
8191: 			$s = "SELECT
8192: 					`id_order`,
8193: 					`client`,
8194: 					`id_order_status`,
8195: 					`is_confirmed`
8196: 				FROM `order` 		
8197: 				WHERE	
8198: 					`id_order` = '" . $id_order . "'
8199: 				LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:8212`
```text
8209: 				return $this->showError('404', 'error', 'user is not author');
8210: 
8211: 			}
8212: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5
8213: 				&& $d['id_order_status'] != 6)
8214: 			{
8215: 				return $this->showError('404', 'error', 'wrong booking state');
8216: 			}		
8217: 			if (!empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking already confirmed');
```

### `archive_17012026_1259/taxi/models/api.php:8213`
```text
8210: 
8211: 			}
8212: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5
8213: 				&& $d['id_order_status'] != 6)
8214: 			{
8215: 				return $this->showError('404', 'error', 'wrong booking state');
8216: 			}		
8217: 			if (!empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking already confirmed');
8218: 
```

### `archive_17012026_1259/taxi/models/api.php:8260`
```text
8257: 
8258: 			$s = "SELECT
8259: 					`order`.`id_order`,
8260: 					`order`.`id_order_status`,
8261: 					od.`id_user`,
8262: 					od.`id_order_driver_status`
8263: 				FROM `order` 
8264: 				LEFT JOIN (
8265: 						SELECT
```

### `archive_17012026_1259/taxi/models/api.php:8284`
```text
8281: 
8282: 			$d = fetch_assoc($q);
8283: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8284: 			if ($d['id_order_status'] != 2) 
8285: 			{
8286: 				return $this->showError('404', 'error', 'wrong booking state');
8287: 			}
8288: 			if (empty($d['id_user'])) 
8289: 			{
```

### `archive_17012026_1259/taxi/models/api.php:8755`
```text
8752: 			$s = "SELECT
8753: 					`order`.`id_order`,
8754: 					`order`.`client`,
8755: 					`order`.`id_order_status`,
8756: 					TIMESTAMPDIFF(SECOND,IF(`order`.`datetime_start_plan` = 0,`order`.`create_datetime`,`order`.`datetime_start_plan`),`order`.`max_waiting_datetime`) as max_waiting_time,
8757: 					MAX(`order_waiting_time`.`index`) as max_index
8758: 				FROM `order` 
8759: 				LEFT JOIN `order_waiting_time` USING (`id_order`)		
8760: 				WHERE	
```

### `archive_17012026_1259/taxi/models/api.php:8775`
```text
8772: 			{
8773: 				return $this->showError('404', 'error', 'user is not author');
8774: 			}
8775: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 5 && $d['id_order_status'] != 6) 
8776: 			{
8777: 				return $this->showError('404', 'error', 'wrong booking state');
8778: 			}	
8779: 
8780: 			$d['max_waiting_time'] = (int)$d['max_waiting_time'];
```

### `archive_17012026_1259/taxi/models/api.php:9318`
```text
9315: 				$s = "SELECT
9316: 						`order`.`id_order`,
9317: 						`order`.`client`,
9318: 						`order`.`id_order_status`,
9319: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9320: 						`order`.`night_time`,
9321: 						`order`.`distance_estimate`,
9322: 						`order`.`id_car_class`,
9323: 						`order`.`from_lat`,
```

### `archive_17012026_1259/taxi/models/api.php:9365`
```text
9362: 
9363: 				$d['options'] = json_decode($d['options'],true);
9364: 
9365: 				switch ($d['id_order_status']) {
9366: 					case '1':
9367: 						$allowed_fields = array(
9368: 												'b_start_address'			=>		'from',				
9369: 												'b_start_latitude'			=>		'from_lat',
9370: 												'b_start_longitude'			=>		'from_lng',
```

### `archive_17012026_1259/taxi/models/api.php:9380`
```text
9377: 					case '2':
9378: 						if ($user_service_profile == 'auction')
9379: 						{
9380: 							return $this->showError('404', 'error', "{$d['id_order_status']} is wrong booking state");
9381: 						
9382: 						}
9383: 						$allowed_fields = array(
9384: 												'b_destination_address'		=>		'to',
9385: 												'b_destination_latitude'	=>		'to_lat',
```

### `archive_17012026_1259/taxi/models/api.php:9941`
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
```

### `archive_17012026_1259/taxi/models/api.php:10003`
```text
10000: 
10001: 				$d['c_options'] = json_decode($d['c_options'],true);
10002: 				
10003: 				switch ($d['id_order_status']) {
10004: 					case '1':
10005: 						$allowed_fields = array(
10006: 												'b_start_address'			=>		'from',	
10007: 												'b_start_latitude'			=>		'from_lat',
10008: 												'b_start_longitude'			=>		'from_lng',
```

### `archive_17012026_1259/taxi/models/api.php:11219`
```text
11216: 				$s = "SELECT
11217: 						`id_order`,
11218: 						`client`,
11219: 						`id_order_status`,
11220: 						`tips`
11221: 					FROM `order` 		
11222: 					WHERE	
11223: 						`id_order` = '" . $id_order . "'
11224: 					LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:11232`
```text
11229: 
11230: 				$d = fetch_assoc($q);
11231: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
11232: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
11233: 				if ($d['client'] != $_SESSION[UID]) 
11234: 				{
11235: 					return $this->showError('404', 'error', 'user is not author');
11236: 				}
11237: 				if ($d['tips'] !== NULL) return $this->showError('404', 'error', 'c_tips already inputed');
```

### `archive_17012026_1259/taxi/models/api.php:11267`
```text
11264: 
11265: 				$s = "SELECT
11266: 						`order`.`id_order`,
11267: 						`order`.`id_order_status`,
11268: 						od.`id_user`,
11269: 						od.`id_order_driver_status`,
11270: 						od.`tips`
11271: 					FROM `order`
11272: 					LEFT JOIN (
```

### `archive_17012026_1259/taxi/models/api.php:11293`
```text
11290: 
11291: 				$d = fetch_assoc($q);
11292: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
11293: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
11294: 
11295: 				if (empty($d['id_user'])) 
11296: 				{
11297: 					return $this->showError('404', 'error', 'user is not performer');
11298: 				}
```

### `archive_17012026_1259/taxi/models/api.php:12952`
```text
12949: 								`order_trip`.`id_order`,
12950: 								`order_trip`.`offer_order_datetime`,
12951: 								`order_trip`.`select_trip_datetime`,
12952: 								`order`.`id_order_status`,
12953: 								IFNULL(od.`id_order_driver_status`,0x02)
12954: 							)
12955: 						SEPARATOR 0x01) 
12956: 					) as orders
12957: 				FROM `trip`
```

### `archive_17012026_1259/taxi/models/api.php:13001`
```text
12998: 						$d['orders'][$key]['id_order'],
12999: 						$d['orders'][$key]['offer_order_datetime'],
13000: 						$d['orders'][$key]['select_trip_datetime'],
13001: 						$d['orders'][$key]['id_order_status'],
13002: 						$d['orders'][$key]['id_order_driver_status']
13003: 						)= explode(chr(0),$value);
13004: 
13005: 					if ($d['orders'][$key]['id_order_driver_status'] === chr(2))
13006: 					{
```

### `archive_17012026_1259/taxi/models/api.php:13583`
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
```

### `archive_17012026_1259/taxi/models/api.php:13598`
```text
13595: 
13596: 			$d = fetch_assoc($q);
13597: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
13598: 			if ($d['id_order_status'] == 3 || $d['id_order_status'] == 4) return $this->showError('404', 'error', 'wrong booking state');
13599: 			if ($d['client'] != $_SESSION[UID]) 
13600: 			{
13601: 				return $this->showError('404', 'error', 'user is not author');
13602: 			}
13603: 			if (!empty($d['u_id'])) 
```

### `archive_17012026_1259/taxi/models/api.php:15426`
```text
15423: 				),
15424: 				'booking_states'	=>	array(
15425: 					'table'			=>	'order_status',
15426: 					'id'			=>	'id_order_status',
15427: 					'data_suffix'	=>	'',
15428: 					'allowed_fields'=>	array(
15429: 						'active'	=>		array(
15430: 												'name'	=>	'active',
15431: 												'NULL'	=>	false,
```

### `archive_17012026_1259/taxi/models/api.php:24270`
```text
24267: 					`ticket`.`out_datetime`,
24268: 					`ticket`.`status`,
24269: 					`order`.`pay_datetime` as b_payment_datetime,
24270: 					`order`.`id_order_status` as b_state
24271: 				FROM `ticket`
24272: 				$sql_join
24273: 				LEFT JOIN `order` USING (`id_order`)
24274: 				WHERE 
24275: 					 $sql_where
```

### `archive_17012026_1259/taxi/models/api.php:24570`
```text
24567: 			{
24568: 				$s = "SELECT
24569: 						`order`.`client` as u_id,
24570: 						COUNT(`order`.`id_order_status`) as 'all',
24571: 						COUNT(IF(`order`.`id_order_status` = 1,1,NULL)) as processing,
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
```

### `archive_17012026_1259/taxi/models/api.php:24571`
```text
24568: 				$s = "SELECT
24569: 						`order`.`client` as u_id,
24570: 						COUNT(`order`.`id_order_status`) as 'all',
24571: 						COUNT(IF(`order`.`id_order_status` = 1,1,NULL)) as processing,
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
```

### `archive_17012026_1259/taxi/models/api.php:24572`
```text
24569: 						`order`.`client` as u_id,
24570: 						COUNT(`order`.`id_order_status`) as 'all',
24571: 						COUNT(IF(`order`.`id_order_status` = 1,1,NULL)) as processing,
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
24577: 					FROM `order`
```

### `archive_17012026_1259/taxi/models/api.php:24573`
```text
24570: 						COUNT(`order`.`id_order_status`) as 'all',
24571: 						COUNT(IF(`order`.`id_order_status` = 1,1,NULL)) as processing,
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
24577: 					FROM `order`
24578: 					WHERE
```

### `archive_17012026_1259/taxi/models/api.php:24574`
```text
24571: 						COUNT(IF(`order`.`id_order_status` = 1,1,NULL)) as processing,
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
24577: 					FROM `order`
24578: 					WHERE
24579: 						`order`.`client` in (" . implode(',',array_keys($stat)) . ")
```

### `archive_17012026_1259/taxi/models/api.php:24575`
```text
24572: 						COUNT(IF(`order`.`id_order_status` = 2,1,NULL)) as approved,
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
24577: 					FROM `order`
24578: 					WHERE
24579: 						`order`.`client` in (" . implode(',',array_keys($stat)) . ")
24580: 					GROUP BY
```

### `archive_17012026_1259/taxi/models/api.php:24576`
```text
24573: 						COUNT(IF(`order`.`id_order_status` = 3,1,NULL)) as canceled,
24574: 						COUNT(IF(`order`.`id_order_status` = 4,1,NULL)) as completed,
24575: 						COUNT(IF(`order`.`id_order_status` = 5,1,NULL)) as pending,
24576: 						COUNT(IF(`order`.`id_order_status` = 6,1,NULL)) as offered
24577: 					FROM `order`
24578: 					WHERE
24579: 						`order`.`client` in (" . implode(',',array_keys($stat)) . ")
24580: 					GROUP BY
24581: 						`order`.`client`
```

### `archive_17012026_1259/taxi/models/m_functions.php:1729`
```text
1726: 		if (empty($data_private['_empty']['order_props'])) unset($data_private['_empty']['order_props']);	
1727: 
1728: 		$s = "SELECT 
1729: 				`id_order_status` as id,
1730: 				" . sql_for_langs($data['langs'], 'name_', '') . "
1731: 			FROM `order_status`
1732: 			WHERE
1733: 				`active` = '1'
1734: 			";
```

### `archive_17012026_1259/taxi/config/stripe_monitor.php:18`
```text
15: 			`id_order`
16: 		FROM `order` 
17: 		WHERE
18: 			 `confirm_limit_datetime` !=0 AND now() > `confirm_limit_datetime` AND `pay_datetime` = 0 AND `id_order_status` != 3
19: 		";
20: 
21: 	$q = query($s);
22: 	if ($q === false) break;
23: 	$order_succeeded = $order_failed = array();
```

### `archive_17012026_1259/taxi/config/system_bot.php:16`
```text
13: 	{
14: 		$s = "UPDATE `order`
15: 			SET 
16: 				`id_order_status` = '1',
17: 				`process_datetime` = '" . $now_datetime . "',
18: 				`last_edit_datetime` = '" . $now_datetime . "',
19: 				`last_edit_user` = '0'
20: 			WHERE
21: 				#`id_order`.`id_order_status` in (5,6)
```

### `archive_17012026_1259/taxi/config/system_bot.php:21`
```text
18: 				`last_edit_datetime` = '" . $now_datetime . "',
19: 				`last_edit_user` = '0'
20: 			WHERE
21: 				#`id_order`.`id_order_status` in (5,6)
22: 				`id_order`.`id_order_status` = 5
23: 			";
24: 
25: 		$q = query($s);
26: 		if ($q === false)  file_put_contents($log, "$now_datetime : empty d_s_bot - database update failed\n", FILE_APPEND);
```

### `archive_17012026_1259/taxi/config/system_bot.php:22`
```text
19: 				`last_edit_user` = '0'
20: 			WHERE
21: 				#`id_order`.`id_order_status` in (5,6)
22: 				`id_order`.`id_order_status` = 5
23: 			";
24: 
25: 		$q = query($s);
26: 		if ($q === false)  file_put_contents($log, "$now_datetime : empty d_s_bot - database update failed\n", FILE_APPEND);
27: 		return true;
```

### `archive_17012026_1259/taxi/config/system_bot.php:269`
```text
266: 	{
267: 		$s = "UPDATE `order`
268: 			SET 
269: 				`id_order_status` = '1',
270: 				`process_datetime` = '" . $now_datetime . "',
271: 				`last_edit_datetime` = '" . $now_datetime . "',
272: 				`last_edit_user` = '0'
273: 			WHERE
274: 				`id_order`.`id_order_status` in (5,6)
```

### `archive_17012026_1259/taxi/config/system_bot.php:274`
```text
271: 				`last_edit_datetime` = '" . $now_datetime . "',
272: 				`last_edit_user` = '0'
273: 			WHERE
274: 				`id_order`.`id_order_status` in (5,6)
275: 			";
276: 
277: 		$q = query($s);
278: 		if ($q === false) file_put_contents($log, "$now_datetime : empty max_arr_len - database update failed\n", FILE_APPEND);
279: 		return true;
```

### `archive_17012026_1259/taxi/config/system_bot.php:318`
```text
315: 			{		
316: 				$s = "UPDATE `order`
317: 					SET 
318: 						`id_order_status` = '1',
319: 						`process_datetime` = '" . $now_datetime . "',
320: 						`last_edit_datetime` = '" . $now_datetime . "',
321: 						`last_edit_user` = '0'
322: 					WHERE
323: 						`id_order`.`id_order_status` in (5,6)
```

### `archive_17012026_1259/taxi/config/system_bot.php:323`
```text
320: 						`last_edit_datetime` = '" . $now_datetime . "',
321: 						`last_edit_user` = '0'
322: 					WHERE
323: 						`id_order`.`id_order_status` in (5,6)
324: 					";
325: 
326: 				$q = query($s);
327: 				if ($q === false) file_put_contents($log, "$now_datetime : max_arr_len=1 - database update to processing failed\n", FILE_APPEND);
328: 				
```

### `archive_17012026_1259/taxi/config/system_bot.php:332`
```text
329: 				
330: 				$s = "UPDATE `order`
331: 					SET 
332: 						`id_order_status` = '3',
333: 						`cancel_datetime` = '" . $now_datetime . "',
334: 						`last_edit_datetime` = '" . $now_datetime . "',
335: 						`last_edit_user` = '0'
336: 					WHERE
337: 						`id_order` = '1' AND 
```

### `archive_17012026_1259/taxi/config/system_bot.php:349`
```text
346: 			{
347: 				$s = "UPDATE `order`
348: 					SET 
349: 						`id_order_status` = '3',
350: 						`cancel_datetime` = '" . $now_datetime . "',
351: 						`last_edit_datetime` = '" . $now_datetime . "',
352: 						`last_edit_user` = '0'
353: 					WHERE
354: 						`id_order` = '1' AND 
```

### `archive_17012026_1259/taxi/config/system_bot.php:367`
```text
364: 
365: 	$s = "SELECT
366: 			`order`.`id_order`,
367: 			`order`.`id_order_status`,
368: 			`order`.`from`,
369: 			`order`.`from_lat`,
370: 			`order`.`from_lng`,
371: 			TIMESTAMPDIFF(SECOND,NOW(),`order`.`datetime_start_plan`) as start_interval,
372: 			`order`.`id_car_class`,
```

### `archive_17012026_1259/taxi/config/system_bot.php:379`
```text
376: 		FROM `order` 
377: 		LEFT JOIN `users_favorite` ON `users_favorite`.`id_user` = `order`.`client`
378: 		WHERE	
379: 			`order`.`id_order_status` = '5'" . $sql_where_code . " AND `order`.`datetime_start_plan` AND `order`.`datetime_start_plan`<= now() + INTERVAL '" . $max_start_select_interval . "' SECOND
380: 		GROUP BY 
381: 			`order`.`id_order`
382: 		";
383: 
384: 	$q = query($s);
```

### `archive_17012026_1259/taxi/config/system_bot.php:428`
```text
425: 
426: 	$s = "SELECT
427: 			`order`.`id_order`,
428: 			`order`.`id_order_status`,
429: 			`order`.`from`,
430: 			`order`.`from_lat`,
431: 			`order`.`from_lng`,
432: 			`order`.`id_car_class`,
433: 			(SELECT
```

### `archive_17012026_1259/taxi/config/system_bot.php:451`
```text
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

### `archive_17012026_1259/taxi/config/system_bot.php:589`
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:890`
```text
887: 	{
888: 		$s = "UPDATE `order`
889: 			SET 
890: 				`id_order_status` = '6',
891: 				`offer_datetime` = '" . $now_datetime . "',
892: 				`last_edit_datetime` = '" . $now_datetime . "',
893: 				`last_edit_user` = '0'
894: 			WHERE
895: 				`id_order` in (" . implode(",", $order_for_offer) . ")
```

### `archive_17012026_1259/taxi/config/system_bot.php:906`
```text
903: 	{
904: 		$s = "UPDATE `order`
905: 			SET 
906: 				`id_order_status` = '1',
907: 				`process_datetime` = '" . $now_datetime . "',
908: 				`last_edit_datetime` = '" . $now_datetime . "',
909: 				`last_edit_user` = '0'
910: 			WHERE
911: 				`id_order` in (" . implode(",", $order_for_process) . ")
```

### `archive_17012026_1259/taxi/config/system_bot.php:922`
```text
919: 	{
920: 		$s = "UPDATE `order`
921: 			SET 
922: 				`id_order_status` = '3',
923: 				`cancel_datetime` = '" . $now_datetime . "',
924: 				`last_edit_datetime` = '" . $now_datetime . "',
925: 				`last_edit_user` = '0'
926: 			WHERE
927: 				`id_order` in (" . implode(",", $order_for_cancel) . ")
```

### `archive_17012026_1259/taxi/controllers/c_edit_langs.php:686`
```text
683: 										)
684: 				),
685: 				'table'			=>	'order_status',
686: 				'key'			=>	'id_order_status',
687: 				'type'			=>	'1'
688: 			 ),
689: 			'outer_script_templates' => 	array(
690: 				'title' 		=> 'Внешние шаблоны скриптов',
691: 				'fields'		=>	array(
```

### `archive_17012026_1259/taxi/controllers/c_stripe.php:440`
```text
437: 							`pay_datetime` = " . ($status == 'succeeded' ? 'now()' : '`pay_datetime`') . ", 
438: 							`last_edit_datetime` = now(),
439: 							`last_edit_user` = '0'
440: 							" . ($status == 'failed' ? ",`id_order_status` = '3'" : "") . "
441: 						WHERE
442: 							`id_order` = '" . $order . "'
443: 						";
444: 
445: 					$q = query($s);
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:158`
```text
155: 								}
156: 								if ($_POST['action'] == 'set_performer')
157: 								{
158: 									$out = $API->setDriver(isset($_POST['data'])?$_POST['data']:'',trim($_GET['par'][3]),!empty($_POST['u_id'])?trim($_POST['u_id']):'',empty($_POST['performer'])?0:1,isset($_POST['b_driver_code'])?trim($_POST['b_driver_code']):NULL,isset($_POST['t_id'])?trim($_POST['t_id']):'');
159: 								}
160: 								elseif ($_POST['action'] == 'set_arrive_state')
161: 								{
162: 									$out = $API->setCarIsArrived(trim($_GET['par'][3]));
163: 								}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:166`
```text
163: 								}
164: 								if ($_POST['action'] == 'set_performer')
165: 								{
166: 									$out = $API->setDriver(isset($_POST['data'])?$_POST['data']:'',trim($_GET['par'][3]),!empty($_POST['u_id'])?trim($_POST['u_id']):'',empty($_POST['performer'])?0:1,isset($_POST['b_driver_code'])?trim($_POST['b_driver_code']):NULL,isset($_POST['t_id'])?trim($_POST['t_id']):'');
167: 								}
168: 								elseif ($_POST['action'] == 'set_arrive_state')
169: 								{
170: 									$out = $API->setCarIsArrived(trim($_GET['par'][3]));
171: 								}
```

## 3. Position Evidence

Найдено position-related contexts: **27**.
### `archive_17012026_1259/taxi/models/api.php:4586`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4587`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4588`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4590`
```text
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
4594: 								)
4595: 							) SEPARATOR 0x01)
```

### `archive_17012026_1259/taxi/models/api.php:4609`
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
```

### `archive_17012026_1259/taxi/models/api.php:4611`
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
```

### `archive_17012026_1259/taxi/models/api.php:4657`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4658`
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
```

### `archive_17012026_1259/taxi/models/api.php:4659`
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
```

### `archive_17012026_1259/taxi/models/api.php:4661`
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
```

### `archive_17012026_1259/taxi/models/api.php:4683`
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
```

### `archive_17012026_1259/taxi/models/api.php:4685`
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
```

### `archive_17012026_1259/taxi/models/api.php:5071`
```text
5068: 			{
5069: 				$s = "SELECT 	
5070: 						`users`.`id_user`,
5071: 						`users_location`.`lat`,
5072: 						`users_location`.`lng`,
5073: 						`car`.`id_car_class`,
5074: 						(SELECT
5075: 							GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
5076: 						 FROM `users_order_comment`
```

### `archive_17012026_1259/taxi/models/api.php:5072`
```text
5069: 				$s = "SELECT 	
5070: 						`users`.`id_user`,
5071: 						`users_location`.`lat`,
5072: 						`users_location`.`lng`,
5073: 						`car`.`id_car_class`,
5074: 						(SELECT
5075: 							GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
5076: 						 FROM `users_order_comment`
5077: 						 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
```

### `archive_17012026_1259/taxi/models/api.php:5100`
```text
5097: 							NULL) SEPARATOR ';')
5098: 						) as license
5099: 					FROM `users`
5100: 					LEFT JOIN `users_location` USING (`id_user`)
5101: 					LEFT JOIN `car_users` ON `car_users`.`id_user` = `users`.`id_user`
5102: 					LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`				
5103: 					LEFT JOIN `taxi_license` ON `taxi_license`.`id_car` = `car`.`id_car`	
5104: 					LEFT JOIN `taxi_license_order_location` ON `taxi_license_order_location`.`id_taxi_license` = `taxi_license`.`id_taxi_license`
5105: 					WHERE
```

### `archive_17012026_1259/taxi/models/api.php:7757`
```text
7754: 								`order_driver`.`id_user`,
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:7758`
```text
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7759`
```text
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7787`
```text
7784: 				FROM `order` 
7785: 				" . $sql_left_join . "
7786: 				LEFT JOIN `order_driver` USING (`id_order`)
7787: 				LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
7788: 				WHERE	
7789: 					" . $sql_where . "
7790: 				GROUP BY `order`.`id_order`
7791: 				ORDER BY `order`.`last_edit_datetime` DESC
7792: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
```

### `archive_17012026_1259/taxi/models/api.php:8164`
```text
8161: 				return $this->showError('404', 'error', 'unauthorized access');
8162: 			}
8163: 
8164: 			$s = "REPLACE INTO `users_location`
8165: 				SET 
8166: 					`id_user` = '" . $_SESSION[UID]  . "',
8167: 					`lat` = '" . $lat  . "',
8168: 					`lng` = '" . $lng  . "',
8169: 					`datetime` = now()
```

### `archive_17012026_1259/taxi/config/system_bot.php:551`
```text
548: 		}
549: 		else
550: 		{
551: 			$sql_where_loc = "`users_location`.`datetime` > NOW() - INTERVAL '" . $loc_sec . "' SECOND AND";
552: 		}
553: 		$sql_where_car_users = "";//"AND `car_users`.`used` = '1'";
554: 		$s = "SELECT 
555: 				`users`.`id_user`,
556: 				`users`.`referrer`,
```

### `archive_17012026_1259/taxi/config/system_bot.php:561`
```text
558: 				`users`.`s_rating`,
559: 				`users`.`u_max_rating`,
560: 				`users`.`u_rating`,			
561: 				`users_location`.`lat`,
562: 				`users_location`.`lng`,
563: 				`car`.`id_car_class`
564: 			FROM `users`
565: 			LEFT JOIN `users_location` USING (`id_user`)
566: 			LEFT JOIN `car_users` USING (`id_user`)
```

### `archive_17012026_1259/taxi/config/system_bot.php:562`
```text
559: 				`users`.`u_max_rating`,
560: 				`users`.`u_rating`,			
561: 				`users_location`.`lat`,
562: 				`users_location`.`lng`,
563: 				`car`.`id_car_class`
564: 			FROM `users`
565: 			LEFT JOIN `users_location` USING (`id_user`)
566: 			LEFT JOIN `car_users` USING (`id_user`)
567: 			LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`
```

### `archive_17012026_1259/taxi/config/system_bot.php:565`
```text
562: 				`users_location`.`lng`,
563: 				`car`.`id_car_class`
564: 			FROM `users`
565: 			LEFT JOIN `users_location` USING (`id_user`)
566: 			LEFT JOIN `car_users` USING (`id_user`)
567: 			LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`
568: 			WHERE
569: 				`id_role` = '2' AND `id_verification_status` = '2' AND `deleted` = '0' AND `active` = '1' AND
570: 				`out_order` = '0' AND 
```

### `archive_17012026_1259/taxi/config/system_bot.php:572`
```text
569: 				`id_role` = '2' AND `id_verification_status` = '2' AND `deleted` = '0' AND `active` = '1' AND
570: 				`out_order` = '0' AND 
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
573: 				(
574: 					SELECT 
575: 						`id_user`
576: 					FROM `users_ban`
577: 					WHERE
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2397`
```text
2394: 			</form>
2395: 		</fieldset>
2396: 		<pre>
2397: 	https://ibronevik.ru/taxi/api/v1/location								POST
2398: 		 Запись координат авторизированного пользователя в базу.
2399: 			Доступно только для авторизованного пользователя.
2400: 			Параметры запроса;
2401: 				latitude			широта				необходимо
2402: 				longitude			долгота				необходимо
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2406`
```text
2403: 			Ответ сервера:
2404: 			{'code':'200','status':'success'}</pre>
2405: 		<fieldset class="form"><legend title="Записывает координаты авторизированного пользователя в базу. Доступна только для авторизованного пользователя.">Изменение координат пользователя</legend>
2406: 			<form class="complex" action="api/v1/location" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
```

## 4. Что можно подтвердить

В backend source corpus независимо подтверждаются два механизма:

### Assignment

```text
setDriver
   ↓
Order / order_driver state
   ↓
assigned performer relation```

### Position

```text
User
   ↓
users_location / location API   ↓
Position```

Это **не означает автоматически**, что:

```text
Order
   → CURRENT_ASSIGNED_DRIVER_POSITION```

поскольку это требует Evidence, связывающего конкретного assigned performer с конкретным position read/exposure path.

## 5. Ключевой methodological boundary

Нельзя получить из двух независимых Claims третий Claim простым логическим сложением:

```text
Claim A:
Order → assigned User

Claim B:
User → current Position

НЕ ДОКАЗАНО автоматически:
Order → assigned User → current Position```

Последний relation требует либо прямого consumer code, либо другого Evidence, показывающего, что assignment используется при выборе Position.

## 6. Current graph layer

Допустимо:

```text
Order
  ──ASSIGNED_PERFORMER──>
User

User
  ──HAS_CURRENT_POSITION──>
Position```

Пока не добавляем как CONFIRMED:

```text
Order
  ──EXPOSES_ASSIGNED_PERFORMER_POSITION──>
Position```

## 7. AS-IS / TO-BE boundary

Если frontend должен показывать машину исполнителя на карте, это может быть TO-BE requirement.

Но production graph должен содержать только наблюдаемое AS-IS:

```text
Assignment
    +
User Position
    +
existing exposure path```

Наличие первых двух не доказывает наличие третьего.

## 8. Gap Report

```text
G-22-01  exact assignment → performer User relation       OPEN
G-22-02  exact assigned User → position read relation     OPEN
G-22-03  position exposure to order owner                 OPEN
G-22-04  frontend consumer of assigned performer position OPEN
G-22-05  map rendering path                                OPEN
```

## 9. Следующий шаг

Не расширять поиск по всем position occurrences. Взять конкретный production consumer, который после `setDriver` получает/возвращает order state, и пройти его до поля/объекта performer. Затем отдельно проверить, откуда frontend получает Position.

Целевая цепочка:

```text
setDriver
   ↓
order_driver
   ↓
assigned User
   ↓
position query/exposure
   ↓
frontend consumer
   ↓
map```

Если один из переходов отсутствует в AS-IS, зафиксировать это как GAP, а не достраивать граф.