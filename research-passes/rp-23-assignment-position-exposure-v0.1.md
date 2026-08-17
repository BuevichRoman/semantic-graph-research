# Backend Semantic Graph — Research Pass 23
# Assignment → Position Exposure v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-22 Assignment → User → Position Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Research Question

> Можно ли доказать production-цепочку `Order → assigned User → Position`, и если да — где возникает exposure этой Position потребителю?

RP-22 установил два независимых кластера Evidence, но не доказал их соединение. RP-23 ищет именно мост, а не новые независимые occurrences.

## 2. Поисковая стратегия

Приоритет отдавался contexts, где одновременно встречаются:

```text
order_driver / id_driver / id_user
        +
users_location / selectLocation / setLocation / notifyPosition / position
```

Это позволяет отделить потенциальный bridge от обычных отдельных употреблений `position`.

## 3. Наиболее релевантные contexts

Всего найдено candidate contexts: **180**.
### `archive_17012026_1259/taxi/models/api.php:4580`
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
```

### `archive_17012026_1259/taxi/models/api.php:4581`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4583`
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
```

### `archive_17012026_1259/taxi/models/api.php:4584`
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
```

### `archive_17012026_1259/taxi/models/api.php:4585`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4586`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4587`
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
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
4594: 								)
```

### `archive_17012026_1259/taxi/models/api.php:4608`
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
```

### `archive_17012026_1259/taxi/models/api.php:4609`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4651`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4654`
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
```

### `archive_17012026_1259/taxi/models/api.php:4655`
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
```

### `archive_17012026_1259/taxi/models/api.php:4656`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4657`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4658`
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
4663: 									`order_driver`.`start_datetime`,
4664: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
4665: 									`order_driver`.`options`,0x02)
```

### `archive_17012026_1259/taxi/models/api.php:4659`
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
```

### `archive_17012026_1259/taxi/models/api.php:4660`
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
```

### `archive_17012026_1259/taxi/models/api.php:4661`
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
```

### `archive_17012026_1259/taxi/models/api.php:4662`
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
```

### `archive_17012026_1259/taxi/models/api.php:4663`
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
```

### `archive_17012026_1259/taxi/models/api.php:4681`
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
```

### `archive_17012026_1259/taxi/models/api.php:4682`
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
```

### `archive_17012026_1259/taxi/models/api.php:4683`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4685`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:7751`
```text
7747: 					`order`.`distance_estimate` as b_distance_estimate,
7748: 					`order`.`price_estimate` as b_price_estimate,
7749: 					`order`.`currency` as b_currency,
7750: 					`order`.`night_time` as b_night,
7751: 					IF(`order_driver`.`id_order` IS NULL,NULL,
7752: 						GROUP_CONCAT(
7753: 							CONCAT_WS(0x00,
7754: 								`order_driver`.`id_user`,
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
```

### `archive_17012026_1259/taxi/models/api.php:7754`
```text
7750: 					`order`.`night_time` as b_night,
7751: 					IF(`order_driver`.`id_order` IS NULL,NULL,
7752: 						GROUP_CONCAT(
7753: 							CONCAT_WS(0x00,
7754: 								`order_driver`.`id_user`,
7755: 								`order_driver`.`id_car`,
7756: 								`order_driver`.`id_order_driver_status`,
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:7755`
```text
7751: 					IF(`order_driver`.`id_order` IS NULL,NULL,
7752: 						GROUP_CONCAT(
7753: 							CONCAT_WS(0x00,
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

### `archive_17012026_1259/taxi/models/api.php:7756`
```text
7752: 						GROUP_CONCAT(
7753: 							CONCAT_WS(0x00,
7754: 								`order_driver`.`id_user`,
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

### `archive_17012026_1259/taxi/models/api.php:7757`
```text
7753: 							CONCAT_WS(0x00,
7754: 								`order_driver`.`id_user`,
7755: 								`order_driver`.`id_car`,
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

### `archive_17012026_1259/taxi/models/api.php:7758`
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
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
7765: 								`order_driver`.`cancel_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7786`
```text
7782: 					 WHERE `order_service`.`id_order` = `order`.`id_order`
7783: 					) as b_services
7784: 				FROM `order` 
7785: 				" . $sql_left_join . "
7786: 				LEFT JOIN `order_driver` USING (`id_order`)
7787: 				LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
7788: 				WHERE	
7789: 					" . $sql_where . "
7790: 				GROUP BY `order`.`id_order`
7791: 				ORDER BY `order`.`last_edit_datetime` DESC
7792: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
7793: 				";
```

### `archive_17012026_1259/taxi/models/api.php:7787`
```text
7783: 					) as b_services
7784: 				FROM `order` 
7785: 				" . $sql_left_join . "
7786: 				LEFT JOIN `order_driver` USING (`id_order`)
7787: 				LEFT JOIN `users_location` ON `users_location`.`id_user` = `order_driver`.`id_user`
7788: 				WHERE	
7789: 					" . $sql_where . "
7790: 				GROUP BY `order`.`id_order`
7791: 				ORDER BY `order`.`last_edit_datetime` DESC
7792: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
7793: 				";
7794: 
```

### `archive_17012026_1259/taxi/models/api.php:4588`
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
4593: 									`order_driver`.`options`
4594: 								)
4595: 							) SEPARATOR 0x01)
```

### `archive_17012026_1259/taxi/models/api.php:4589`
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
4594: 								)
4595: 							) SEPARATOR 0x01)
4596: 						) as drivers,
```

### `archive_17012026_1259/taxi/models/api.php:4590`
```text
4586: 									IFNULL(`users_location`.`lat`,''),
4587: 									IFNULL(`users_location`.`lng`,''),
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
4594: 								)
4595: 							) SEPARATOR 0x01)
4596: 						) as drivers,
4597: 						(SELECT
```

### `archive_17012026_1259/taxi/models/api.php:4591`
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
4596: 						) as drivers,
4597: 						(SELECT
4598: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
```

### `archive_17012026_1259/taxi/models/api.php:4592`
```text
4588: 									IFNULL(`users_location`.`datetime`,''),
4589: 									`order_driver`.`candidacy_datetime`,
4590: 									`order_driver`.`appoint_datetime`,
4591: 									`order_driver`.`arrive_datetime`,
4592: 									`order_driver`.`start_datetime`,
4593: 									`order_driver`.`options`
4594: 								)
4595: 							) SEPARATOR 0x01)
4596: 						) as drivers,
4597: 						(SELECT
4598: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4599: 						 FROM `order_comment_items`
```

### `archive_17012026_1259/taxi/models/api.php:7759`
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
7764: 								`order_driver`.`appoint_datetime`,
7765: 								`order_driver`.`cancel_datetime`,
7766: 								`order_driver`.`arrive_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7761`
```text
7757: 								IFNULL(`users_location`.`lat`,''),
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
7765: 								`order_driver`.`cancel_datetime`,
7766: 								`order_driver`.`arrive_datetime`,
7767: 								`order_driver`.`start_datetime`,
7768: 								`order_driver`.`complete_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:7762`
```text
7758: 								IFNULL(`users_location`.`lng`,''),
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
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

### `archive_17012026_1259/taxi/models/api.php:7763`
```text
7759: 								IFNULL(`users_location`.`datetime`,''),
7760: 								" . $sql_order_driver . "
7761: 								IFNULL(`order_driver`.`cancel_reason`,0x02),
7762: 								IFNULL(`order_driver`.`rating`,0x02),
7763: 								`order_driver`.`candidacy_datetime`,
7764: 								`order_driver`.`appoint_datetime`,
7765: 								`order_driver`.`cancel_datetime`,
7766: 								`order_driver`.`arrive_datetime`,
7767: 								`order_driver`.`start_datetime`,
7768: 								`order_driver`.`complete_datetime`,
7769: 								IFNULL(`order_driver`.`tips`,0x02),
7770: 								" . $sql_c_options . "
```

### `archive_17012026_1259/taxi/models/api.php:8158`
```text
8154: 								)
8155: 			);
8156: 		}
8157: 
8158: 		public function setLocation($lat = "", $lng = "")
8159: 		{
8160: 			if (empty($_SESSION[UID])) {
8161: 				return $this->showError('404', 'error', 'unauthorized access');
8162: 			}
8163: 
8164: 			$s = "REPLACE INTO `users_location`
8165: 				SET 
```

### `archive_17012026_1259/taxi/config/system_bot.php:565`
```text
561: 				`users_location`.`lat`,
562: 				`users_location`.`lng`,
563: 				`car`.`id_car_class`
564: 			FROM `users`
565: 			LEFT JOIN `users_location` USING (`id_user`)
566: 			LEFT JOIN `car_users` USING (`id_user`)
567: 			LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`
568: 			WHERE
569: 				`id_role` = '2' AND `id_verification_status` = '2' AND `deleted` = '0' AND `active` = '1' AND
570: 				`out_order` = '0' AND 
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
```

### `archive_17012026_1259/taxi/config/system_bot.php:566`
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
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
573: 				(
```

### `archive_17012026_1259/taxi/config/system_bot.php:572`
```text
568: 			WHERE
569: 				`id_role` = '2' AND `id_verification_status` = '2' AND `deleted` = '0' AND `active` = '1' AND
570: 				`out_order` = '0' AND 
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
573: 				(
574: 					SELECT 
575: 						`id_user`
576: 					FROM `users_ban`
577: 					WHERE
578: 						`users_ban`.`id_user` = `users`.`id_user` AND (`expire_datetime` = 0 OR `expire_datetime` > NOW()) AND
579: 						(`auth` = '1' OR `order` = '1')
```

### `archive_17012026_1259/taxi/config/system_bot.php:575`
```text
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
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
```

### `archive_17012026_1259/taxi/models/api.php:5070`
```text
5066: 			$union = false;
5067: 			if (isset($filter) && $this->id_role == 2)
5068: 			{
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

### `archive_17012026_1259/taxi/models/api.php:5071`
```text
5067: 			if (isset($filter) && $this->id_role == 2)
5068: 			{
5069: 				$s = "SELECT 	
5070: 						`users`.`id_user`,
5071: 						`users_location`.`lat`,
5072: 						`users_location`.`lng`,
5073: 						`car`.`id_car_class`,
5074: 						(SELECT
5075: 							GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
5076: 						 FROM `users_order_comment`
5077: 						 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
5078: 						 ORDER BY `users_order_comment`.`id_order_comment`
```

### `archive_17012026_1259/taxi/models/api.php:5072`
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
5077: 						 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
5078: 						 ORDER BY `users_order_comment`.`id_order_comment`
5079: 						) as b_comments,
```

### `archive_17012026_1259/taxi/models/api.php:5100`
```text
5096: 								),
5097: 							NULL) SEPARATOR ';')
5098: 						) as license
5099: 					FROM `users`
5100: 					LEFT JOIN `users_location` USING (`id_user`)
5101: 					LEFT JOIN `car_users` ON `car_users`.`id_user` = `users`.`id_user`
5102: 					LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`				
5103: 					LEFT JOIN `taxi_license` ON `taxi_license`.`id_car` = `car`.`id_car`	
5104: 					LEFT JOIN `taxi_license_order_location` ON `taxi_license_order_location`.`id_taxi_license` = `taxi_license`.`id_taxi_license`
5105: 					WHERE
5106: 						`users`.`id_user` = " . $_SESSION[UID]. " AND `car_users`.`used` = '1'
5107: 					GROUP BY `users`.`id_user`
```

### `archive_17012026_1259/taxi/models/api.php:5101`
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
5106: 						`users`.`id_user` = " . $_SESSION[UID]. " AND `car_users`.`used` = '1'
5107: 					GROUP BY `users`.`id_user`
5108: 					LIMIT 1
```

### `archive_17012026_1259/taxi/config/system_bot.php:436`
```text
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:444`
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:597`
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:598`
```text
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

### `archive_17012026_1259/taxi/config/system_bot.php:602`
```text
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:951`
```text
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
```

### `archive_17012026_1259/taxi/config/system_bot.php:953`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4515`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:4521`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4527`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:4531`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4664`
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
```

### `archive_17012026_1259/taxi/models/api.php:4665`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4694`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4697`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:4699`
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
```

### `archive_17012026_1259/taxi/models/api.php:4700`
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
```

### `archive_17012026_1259/taxi/models/api.php:4982`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:4988`
```text
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

### `archive_17012026_1259/taxi/models/api.php:4994`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:4998`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5008`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5011`
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
```

### `archive_17012026_1259/taxi/models/api.php:5012`
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
```

### `archive_17012026_1259/taxi/models/api.php:5013`
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
```

### `archive_17012026_1259/taxi/models/api.php:5014`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5022`
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
```

### `archive_17012026_1259/taxi/models/api.php:5023`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5025`
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
```

### `archive_17012026_1259/taxi/models/api.php:5026`
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
```

### `archive_17012026_1259/taxi/models/api.php:5027`
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
```

### `archive_17012026_1259/taxi/models/api.php:5028`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5036`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5038`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5040`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5047`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5049`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5055`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5642`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5648`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5654`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:5658`
```text
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

### `archive_17012026_1259/taxi/models/api.php:5728`
```text
5724: 						`order`.`distance_estimate` as b_distance_estimate,
5725: 						`order`.`price_estimate` as b_price_estimate,
5726: 						`order`.`currency` as b_currency,
5727: 						`order`.`night_time` as b_night,
5728: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5731`
```text
5727: 						`order`.`night_time` as b_night,
5728: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
5736: 									`order_driver`.`candidacy_datetime`,
5737: 									`order_driver`.`appoint_datetime`,
5738: 									`order_driver`.`cancel_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5732`
```text
5728: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
5736: 									`order_driver`.`candidacy_datetime`,
5737: 									`order_driver`.`appoint_datetime`,
5738: 									`order_driver`.`cancel_datetime`,
5739: 									`order_driver`.`arrive_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5733`
```text
5729: 							GROUP_CONCAT(
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
5736: 									`order_driver`.`candidacy_datetime`,
5737: 									`order_driver`.`appoint_datetime`,
5738: 									`order_driver`.`cancel_datetime`,
5739: 									`order_driver`.`arrive_datetime`,
5740: 									`order_driver`.`start_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5734`
```text
5730: 								CONCAT_WS(0x00,
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
5734: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5735: 									IFNULL(`order_driver`.`rating`,0x02),
5736: 									`order_driver`.`candidacy_datetime`,
5737: 									`order_driver`.`appoint_datetime`,
5738: 									`order_driver`.`cancel_datetime`,
5739: 									`order_driver`.`arrive_datetime`,
5740: 									`order_driver`.`start_datetime`,
5741: 									`order_driver`.`complete_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5735`
```text
5731: 									`order_driver`.`id_user`,
5732: 									`order_driver`.`id_car`,
5733: 									`order_driver`.`id_order_driver_status`,
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

### `archive_17012026_1259/taxi/models/api.php:5815`
```text
5811: 						`order`.`distance_estimate` as b_distance_estimate,
5812: 						`order`.`price_estimate` as b_price_estimate,
5813: 						`order`.`currency` as b_currency,
5814: 						`order`.`night_time` as b_night,
5815: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5816: 							GROUP_CONCAT(
5817: 								CONCAT_WS(0x00,
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5818`
```text
5814: 						`order`.`night_time` as b_night,
5815: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5816: 							GROUP_CONCAT(
5817: 								CONCAT_WS(0x00,
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5819`
```text
5815: 						IF(`order_driver`.`id_order` IS NULL,NULL,
5816: 							GROUP_CONCAT(
5817: 								CONCAT_WS(0x00,
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
```

### `archive_17012026_1259/taxi/models/api.php:5820`
```text
5816: 							GROUP_CONCAT(
5817: 								CONCAT_WS(0x00,
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5821`
```text
5817: 								CONCAT_WS(0x00,
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5822`
```text
5818: 									`order_driver`.`id_user`,
5819: 									`order_driver`.`id_car`,
5820: 									`order_driver`.`id_order_driver_status`,							
5821: 									`order_driver`.`id_payment_method`,
5822: 									IFNULL(`order_driver`.`id_payment_card`,0x02),
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
```

### `archive_17012026_1259/taxi/models/api.php:5827`
```text
5823: 									IFNULL(`order_driver`.`sum`,0x02),
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
```

### `archive_17012026_1259/taxi/models/api.php:5828`
```text
5824: 									`order_driver`.`pay_datetime`,
5825: 									IFNULL(`order_driver`.`cancel_reason`,0x02),
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
```

### `archive_17012026_1259/taxi/models/api.php:5829`
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
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
```

### `archive_17012026_1259/taxi/models/api.php:5830`
```text
5826: 									IFNULL(`order_driver`.`rating`,0x02),
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
```

### `archive_17012026_1259/taxi/models/api.php:5831`
```text
5827: 									`order_driver`.`candidacy_datetime`,
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
5838: 						) as drivers,	
```

### `archive_17012026_1259/taxi/models/api.php:5832`
```text
5828: 									`order_driver`.`appoint_datetime`,
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
5838: 						) as drivers,	
5839: 						(SELECT
```

### `archive_17012026_1259/taxi/models/api.php:5833`
```text
5829: 									`order_driver`.`cancel_datetime`,
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
5838: 						) as drivers,	
5839: 						(SELECT
5840: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
```

### `archive_17012026_1259/taxi/models/api.php:5834`
```text
5830: 									`order_driver`.`arrive_datetime`,
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
5838: 						) as drivers,	
5839: 						(SELECT
5840: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
5841: 						 FROM `order_comment_items`
```

### `archive_17012026_1259/taxi/models/api.php:5835`
```text
5831: 									`order_driver`.`start_datetime`,
5832: 									`order_driver`.`complete_datetime`,
5833: 									IFNULL(`order_driver`.`tips`,0x02),
5834: 									IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
5835: 									`order_driver`.`options`,0x02)
5836: 								)
5837: 							SEPARATOR 0x01) 
5838: 						) as drivers,	
5839: 						(SELECT
5840: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
5841: 						 FROM `order_comment_items`
5842: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
```

### `archive_17012026_1259/taxi/models/api.php:5851`
```text
5847: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
5848: 						) as b_services,
5849: 						od.`id_order_driver_status` as c_state
5850: 					FROM `order`
5851: 					LEFT JOIN `order_driver` as od USING (`id_order`)
5852: 					LEFT JOIN `order_driver` USING (`id_order`)			
5853: 					WHERE
5854: 						(`order`.`id_order_status` in (3,4) OR od.`id_order_driver_status` = '2') AND 
5855: 						od.`id_user` = '" . $_SESSION[UID] . "' 
5856: 					GROUP BY `order`.`id_order`
5857: 					ORDER BY `order`.`last_edit_datetime` DESC
5858: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
```

### `archive_17012026_1259/taxi/models/api.php:5852`
```text
5848: 						) as b_services,
5849: 						od.`id_order_driver_status` as c_state
5850: 					FROM `order`
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

### `archive_17012026_1259/taxi/models/api.php:5855`
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
5860: 			}
5861: 
5862: 			$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:6198`
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
```

### `archive_17012026_1259/taxi/models/api.php:6199`
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
```

### `archive_17012026_1259/taxi/models/api.php:6200`
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
```

### `archive_17012026_1259/taxi/models/api.php:6201`
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
```

### `archive_17012026_1259/taxi/models/api.php:6203`
```text
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

### `archive_17012026_1259/taxi/models/api.php:6253`
```text
6249: 							IF(od.`current_car_count` < `order`.`car_count`-1,`order`.`approve_datetime`,now()),	
6250: 						`order`.`car_count` = (SELECT @c_count := `order`.`car_count`),	
6251: 						`order_driver`.`id_order_driver_status` = 
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

### `archive_17012026_1259/taxi/models/api.php:6255`
```text
6251: 						`order_driver`.`id_order_driver_status` = 
6252: 							IF( od.`current_car_count` < `order`.`car_count`,3,1),
6253: 						`order_driver`.`appoint_datetime` = 
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

### `archive_17012026_1259/taxi/models/api.php:6258`
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
6263: 					";
6264: 				
6265: 				query($s);
```

### `archive_17012026_1259/taxi/models/api.php:6259`
```text
6255: 							`order_driver`.`appoint_datetime`)
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
6263: 					";
6264: 				
6265: 				query($s);
6266: 
```

### `archive_17012026_1259/taxi/models/api.php:6260`
```text
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
6263: 					";
6264: 				
6265: 				query($s);
6266: 
6267: 				global $link;
```

### `archive_17012026_1259/taxi/models/api.php:6261`
```text
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
6263: 					";
6264: 				
6265: 				query($s);
6266: 
6267: 				global $link;
6268: 				$rows = mysqli_affected_rows($link);				
```

### `archive_17012026_1259/taxi/models/api.php:6390`
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
```

### `archive_17012026_1259/taxi/models/api.php:6391`
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
```

### `archive_17012026_1259/taxi/models/api.php:6392`
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
```

### `archive_17012026_1259/taxi/models/api.php:6393`
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
```

### `archive_17012026_1259/taxi/models/api.php:6394`
```text
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

### `archive_17012026_1259/taxi/models/api.php:6396`
```text
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
```

### `archive_17012026_1259/taxi/models/api.php:6401`
```text
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
6401: 						IFNULL(MAX(`order_driver`.`index`),0) as max_index,
6402: 						(SELECT
6403: 							COUNT(`order_driver_attempt`.`id_order`)
6404: 						 FROM `order_driver_attempt`
6405: 						 WHERE 
6406: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
6407: 							`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
6408: 						) as attempt,
```

### `archive_17012026_1259/taxi/models/api.php:6407`
```text
6403: 							COUNT(`order_driver_attempt`.`id_order`)
6404: 						 FROM `order_driver_attempt`
6405: 						 WHERE 
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

### `archive_17012026_1259/taxi/models/api.php:6411`
```text
6407: 							`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
6408: 						) as attempt,
6409: 						IF(`order`.`id_order_status` = '6',(
6410: 							SELECT
6411: 								`order_driver_select`.`id_user`
6412: 							 FROM `order_driver_select`
6413: 							 WHERE 
6414: 								`order_driver_select`.`id_order` = `order`.`id_order` AND 
6415: 								`order_driver_select`.`id_user`= '" . $_SESSION[UID] . "'
6416: 						),NULL) as selected,
6417: 						`order`.`night_time`,
6418: 						`order`.`distance_estimate`,
```

### `archive_17012026_1259/taxi/models/api.php:6415`
```text
6411: 								`order_driver_select`.`id_user`
6412: 							 FROM `order_driver_select`
6413: 							 WHERE 
6414: 								`order_driver_select`.`id_order` = `order`.`id_order` AND 
6415: 								`order_driver_select`.`id_user`= '" . $_SESSION[UID] . "'
6416: 						),NULL) as selected,
6417: 						`order`.`night_time`,
6418: 						`order`.`distance_estimate`,
6419: 						`order`.`id_car_class`,
6420: 						`order`.`options`
6421: 					FROM `order` 
6422: 					LEFT JOIN `order_driver` USING (`id_order`)				
```

### `archive_17012026_1259/taxi/models/api.php:6459`
```text
6455: 					{
6456: 						$s = "INSERT INTO `order_driver_attempt`
6457: 							SET 
6458: 								`id_order` = '" . $id_order . "',
6459: 								`id_user` = '" . $_SESSION[UID]  . "',
6460: 								`datetime` = now()
6461: 							";
6462: 
6463: 						$q = query($s);
6464: 						if ($q === false) return $this->showError('404', 'error', 'database attempt insert failed');
6465: 						$q = query("COMMIT");
6466: 						if ($q === false) return $this->showError('404', 'error', 'commit failed');
```

### `archive_17012026_1259/taxi/models/api.php:6638`
```text
6634: 							$s[] = "`" . $key . "` = '" . $value . "'";
6635: 						}
6636: 					}	
6637: 					$index = $d['max_index'] + 1;
6638: 					$s = "INSERT INTO `order_driver`
6639: 						SET 
6640: 							" . implode(",\n", $s) .",
6641: 							`id_order` = '" . $id_order . "',
6642: 							`id_user` = '" . $_SESSION[UID]  . "',
6643: 							`index` = '" . $index . "',
6644: 							`id_order_driver_status` = '" . $id_order_driver_status  . "',
6645: 							`" . $datetime_prefix . "` = now()
```

### `archive_17012026_1259/taxi/models/api.php:6642`
```text
6638: 					$s = "INSERT INTO `order_driver`
6639: 						SET 
6640: 							" . implode(",\n", $s) .",
6641: 							`id_order` = '" . $id_order . "',
6642: 							`id_user` = '" . $_SESSION[UID]  . "',
6643: 							`index` = '" . $index . "',
6644: 							`id_order_driver_status` = '" . $id_order_driver_status  . "',
6645: 							`" . $datetime_prefix . "` = now()
6646: 						";
6647: 
6648: 					$q = query($s);
6649: 					if ($q === false) return $this->showError('404', 'error', 'database insert failed');
```

### `archive_17012026_1259/taxi/models/api.php:6655`
```text
6651: 					$out = array('c_index' => $index,'current_cars_count' => $d['current_car_count'],'b_cars_count' => $d['car_count']);
6652: 				}
6653: 				else
6654: 				{
6655: 					$s = "UPDATE `order_driver`
6656: 						SET 
6657: 							`id_order_driver_status` = '" . $id_order_driver_status  . "',
6658: 							`" . $datetime_prefix . "` = now()
6659: 						WHERE
6660: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "' AND 
6661: 							`id_order_driver_status` = '1'
6662: 						";
```

### `archive_17012026_1259/taxi/models/api.php:6660`
```text
6656: 						SET 
6657: 							`id_order_driver_status` = '" . $id_order_driver_status  . "',
6658: 							`" . $datetime_prefix . "` = now()
6659: 						WHERE
6660: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "' AND 
6661: 							`id_order_driver_status` = '1'
6662: 						";
6663: 					
6664: 					query($s);
6665: 
6666: 					if (mysqli_affected_rows($link) === -1) 
6667: 					{
```

### `archive_17012026_1259/taxi/models/api.php:6812`
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
6817: 							`id_order`,
6818: 							`id_user`,
6819: 							`id_order_driver_status`
```

### `archive_17012026_1259/taxi/models/api.php:6818`
```text
6814: 				FROM `order` 
6815: 				LEFT JOIN (
6816: 						SELECT
6817: 							`id_order`,
6818: 							`id_user`,
6819: 							`id_order_driver_status`
6820: 						FROM
6821: 							`order_driver`
6822: 						WHERE
6823: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
6824: 					) od USING (`id_order`)				
6825: 				WHERE	
```

### `archive_17012026_1259/taxi/models/api.php:6821`
```text
6817: 							`id_order`,
6818: 							`id_user`,
6819: 							`id_order_driver_status`
6820: 						FROM
6821: 							`order_driver`
6822: 						WHERE
6823: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
6824: 					) od USING (`id_order`)				
6825: 				WHERE	
6826: 					`order`.`id_order` = '" . $id_order . "'
6827: 				LIMIT 1
6828: 				";
```

### `archive_17012026_1259/taxi/models/api.php:6823`
```text
6819: 							`id_order_driver_status`
6820: 						FROM
6821: 							`order_driver`
6822: 						WHERE
6823: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
6824: 					) od USING (`id_order`)				
6825: 				WHERE	
6826: 					`order`.`id_order` = '" . $id_order . "'
6827: 				LIMIT 1
6828: 				";
6829: 
6830: 			$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:6840`
```text
6836: 				&& $d['id_order_status'] != 6) 
6837: 			{
6838: 				return $this->showError('404', 'error', 'wrong booking state');
6839: 			}
6840: 			if (empty($d['id_user'])) 
6841: 			{
6842: 				return $this->showError('404', 'error', 'user is not performer');
6843: 			}		
6844: 			if ($d['id_order_driver_status'] == 1) 
6845: 			{
6846: 				return $this->showError('404', 'error', 'not appointed performer');
6847: 			}
```

### `archive_17012026_1259/taxi/models/api.php:6869`
```text
6865: 			$s = "UPDATE `order`,`order_driver`
6866: 				SET 
6867: 					`order`.`last_edit_datetime` = now(),
6868: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6869: 					`order_driver`.`id_order_driver_status` = '4',
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
```

### `archive_17012026_1259/taxi/models/api.php:6870`
```text
6866: 				SET 
6867: 					`order`.`last_edit_datetime` = now(),
6868: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6869: 					`order_driver`.`id_order_driver_status` = '4',
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
6877: 
```

### `archive_17012026_1259/taxi/models/api.php:6872`
```text
6868: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6869: 					`order_driver`.`id_order_driver_status` = '4',
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
6877: 
6878: 			query($s);
6879: 	
```

### `archive_17012026_1259/taxi/models/api.php:6873`
```text
6869: 					`order_driver`.`id_order_driver_status` = '4',
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
6877: 
6878: 			query($s);
6879: 	
6880: 			global $link;
```

### `archive_17012026_1259/taxi/models/api.php:6874`
```text
6870: 					`order_driver`.`arrive_datetime` = now()
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
6877: 
6878: 			query($s);
6879: 	
6880: 			global $link;
6881: 			if (mysqli_affected_rows($link) === -1) 
```

### `archive_17012026_1259/taxi/models/api.php:6875`
```text
6871: 				WHERE
6872: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
6873: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
6874: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
6875: 					`order_driver`.`id_order_driver_status` = '3'
6876: 				";
6877: 
6878: 			query($s);
6879: 	
6880: 			global $link;
6881: 			if (mysqli_affected_rows($link) === -1) 
6882: 			{
```

### `archive_17012026_1259/taxi/models/api.php:6954`
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
```

### `archive_17012026_1259/taxi/models/api.php:6955`
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
```

### `archive_17012026_1259/taxi/models/api.php:6956`
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
```

### `archive_17012026_1259/taxi/models/api.php:6957`
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
```

### `archive_17012026_1259/taxi/models/api.php:6958`
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
```

### `archive_17012026_1259/taxi/models/api.php:6959`
```text
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

### `archive_17012026_1259/taxi/models/api.php:6998`
```text
6994: 					$s = "UPDATE `order`,`order_driver`
6995: 						SET 
6996: 							`order`.`last_edit_datetime` = now(),
6997: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6998: 							`order_driver`.`id_order_driver_status` = '6',
6999: 							`order_driver`.`complete_datetime` = now()
7000: 						WHERE
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
```

### `archive_17012026_1259/taxi/models/api.php:6999`
```text
6995: 						SET 
6996: 							`order`.`last_edit_datetime` = now(),
6997: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
6998: 							`order_driver`.`id_order_driver_status` = '6',
6999: 							`order_driver`.`complete_datetime` = now()
7000: 						WHERE
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
7006: 						";
```

### `archive_17012026_1259/taxi/models/api.php:7002`
```text
6998: 							`order_driver`.`id_order_driver_status` = '6',
6999: 							`order_driver`.`complete_datetime` = now()
7000: 						WHERE
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
7006: 						";
7007: 
7008: 					query($s);
7009: 					
```

### `archive_17012026_1259/taxi/models/api.php:7003`
```text
6999: 							`order_driver`.`complete_datetime` = now()
7000: 						WHERE
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
7006: 						";
7007: 
7008: 					query($s);
7009: 					
7010: 					if (mysqli_affected_rows($link) === -1) 
```

### `archive_17012026_1259/taxi/models/api.php:7004`
```text
7000: 						WHERE
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
7006: 						";
7007: 
7008: 					query($s);
7009: 					
7010: 					if (mysqli_affected_rows($link) === -1) 
7011: 					{
```

### `archive_17012026_1259/taxi/models/api.php:7005`
```text
7001: 							`order`.`id_order` = '" . $id_order . "' AND 
7002: 							`order`.`id_order` = `order_driver`.`id_order` AND
7003: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7004: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7005: 							`order_driver`.`id_order_driver_status` in (3,4,5)
7006: 						";
7007: 
7008: 					query($s);
7009: 					
7010: 					if (mysqli_affected_rows($link) === -1) 
7011: 					{
7012: 						return $this->showError('404', 'error', 'driver update failed');
```

### `archive_17012026_1259/taxi/models/api.php:7190`
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
7195: 								`id_order`,
7196: 								`id_user`,
7197: 								`id_order_driver_status`
```

### `archive_17012026_1259/taxi/models/api.php:7196`
```text
7192: 					FROM `order` 
7193: 					LEFT JOIN (
7194: 							SELECT
7195: 								`id_order`,
7196: 								`id_user`,
7197: 								`id_order_driver_status`
7198: 							FROM
7199: 								`order_driver`
7200: 							WHERE
7201: 								`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
7202: 						) od USING (`id_order`)	
7203: 						
```

### `archive_17012026_1259/taxi/models/api.php:7199`
```text
7195: 								`id_order`,
7196: 								`id_user`,
7197: 								`id_order_driver_status`
7198: 							FROM
7199: 								`order_driver`
7200: 							WHERE
7201: 								`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
7202: 						) od USING (`id_order`)	
7203: 						
7204: 					WHERE	
7205: 						`order`.`id_order` = '" . $id_order . "'
7206: 					LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:7201`
```text
7197: 								`id_order_driver_status`
7198: 							FROM
7199: 								`order_driver`
7200: 							WHERE
7201: 								`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
7202: 						) od USING (`id_order`)	
7203: 						
7204: 					WHERE	
7205: 						`order`.`id_order` = '" . $id_order . "'
7206: 					LIMIT 1
7207: 					";
7208: 
```

### `archive_17012026_1259/taxi/models/api.php:7219`
```text
7215: 					&& $d['id_order_status'] != 6) 
7216: 				{
7217: 					return $this->showError('404', 'error', 'wrong booking state');
7218: 				}
7219: 				if (empty($d['id_user'])) 
7220: 				{
7221: 					return $this->showError('404', 'error', 'user is not performer');
7222: 				}
7223: 				global $link;
7224: 				if ($d['id_order_driver_status'] == 1) 
7225: 				{
7226: 					$q = query("BEGIN");
```

### `archive_17012026_1259/taxi/models/api.php:7240`
```text
7236: 						SET 
7237: 							`order`.`last_edit_datetime` = now(),
7238: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7239: 							`order_driver`.`id_order_driver_status` = '2',
7240: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7241: 							`order_driver`.`cancel_datetime` = now(),
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
```

### `archive_17012026_1259/taxi/models/api.php:7241`
```text
7237: 							`order`.`last_edit_datetime` = now(),
7238: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7239: 							`order_driver`.`id_order_driver_status` = '2',
7240: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7241: 							`order_driver`.`cancel_datetime` = now(),
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
```

### `archive_17012026_1259/taxi/models/api.php:7242`
```text
7238: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7239: 							`order_driver`.`id_order_driver_status` = '2',
7240: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7241: 							`order_driver`.`cancel_datetime` = now(),
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
```

### `archive_17012026_1259/taxi/models/api.php:7245`
```text
7241: 							`order_driver`.`cancel_datetime` = now(),
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
```

### `archive_17012026_1259/taxi/models/api.php:7246`
```text
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
7253: 					if (mysqli_affected_rows($link) === -1) 
```

### `archive_17012026_1259/taxi/models/api.php:7247`
```text
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
7253: 					if (mysqli_affected_rows($link) === -1) 
7254: 					{
```

### `archive_17012026_1259/taxi/models/api.php:7248`
```text
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
7253: 					if (mysqli_affected_rows($link) === -1) 
7254: 					{
7255: 						return $this->showError('404', 'error', 'driver update failed');
```

### `archive_17012026_1259/taxi/models/api.php:7297`
```text
7293: 									) od
7294: 								) < `order`.`car_count`+1,
7295: 								IF(`order`.`id_order_status` in (1,5,6),`order`.`id_order_status`,1),2),
7296: 							`order_driver`.`id_order_driver_status` = '2',
7297: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7298: 							`order_driver`.`cancel_datetime` = now(),
7299: 							`order_driver`.`not_deleted` = NULL
7300: 						WHERE
7301: 							`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` in (1,2,5,6) AND
7302: 							`order`.`id_order` = `order_driver`.`id_order` AND 
7303: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7304: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
```

## 4. Текущий вывод

Search corpus подтверждает многочисленные отдельные usages:

```text
order_driver / driver identityusers_location / locationlocation API / position processing```

Но semantic bridge должен быть доказан именно через код, где:

```text
assigned driver identity        ↓
used as subject/key of position lookup        ↓
position returned/exposed```

Простое наличие `order_driver` и `users_location` в одном проекте не является достаточным Evidence.

## 5. Что не фиксируем

Не создаём пока:

```text
Order
  → READS_ASSIGNED_USER_POSITION```

и:

```text
Order Owner
  → OBSERVES_ASSIGNED_DRIVER_POSITION```

пока не найден конкретный data/control-flow bridge.

## 6. AS-IS boundary

Если backend только хранит:

```text
order_driver
users_location```

но не использует assignment при position lookup/exposure, это означает:

```text
Assignment = CONFIRMED
Position = CONFIRMED
Assignment → Position = UNKNOWN / SOURCE_GAP or BEHAVIOR_UNRESOLVED```

Именно это должно попасть в граф.

## 7. Research Loop

```text
UNKNOWN
   ↓
RQ: Does assigned driver identity select position?   ↓
Expected Evidence:   order_driver.id_driver       → position query key   ↓
Search   ↓
CONFIRMED / REJECTED / SOURCE_GAP```

## 8. Gap Report

```text
G-23-01  order_driver.id_driver → position lookup key      OPEN
G-23-02  position result → order/client response            OPEN
G-23-03  frontend consumer of exposed driver position       OPEN
G-23-04  map rendering of assigned driver position          OPEN
```

## 9. Следующий шаг

Не продолжать общий поиск по `position`.

Нужно выбрать конкретный endpoint/response, который возвращает `order_driver` или order state клиенту, и пройти его field-by-field:

```text
SQL result
   ↓
driver identity field
   ↓
response DTO / JSON
   ↓
frontend consumer
```

Параллельно взять конкретный frontend call `/location` и проверить, каким `u_id` он вызывается. Если `u_id` выводится из assigned driver из order state, bridge будет закрыт.