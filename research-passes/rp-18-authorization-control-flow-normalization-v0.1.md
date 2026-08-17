# Backend Semantic Graph — Research Pass 18
# Authorization Control-Flow Normalization v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-17 Role × Operation Semantic Matrix v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Цель
Нормализовать целые control-flow ветви вокруг role checks, чтобы отличить ALLOW, REJECT, PRECONDITION и UNKNOWN.

## 2. Подтверждённый role mapping
| ID | Business Role |
|---:|---|
| 1 | Client |
| 2 | Driver |
| 4 | Administrator |
| 5 | Agent |
| 6 | Usher |
| 10 | Usher with extended powers |

Источник: `taxi/cache/data.php`.

## 3. Метод нормализации
```text
role condition
    ↓
condition conjunction
    ↓
branch
    ↓
return / continuation
```

Отдельная строка `id_role != N` без восстановления branch semantics не становится ALLOW.

## 4. Function-level contexts
Всего direct role-check contexts: **161**; уникальных function contexts: **72**.
### `addFavoriteUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `10792-10877`
```php
10792: 		public function addFavoriteUser($id_user = "", $id_favorite = "")
10793: 		{
10794: 			if (empty($_SESSION[UID])) {
10795: 				return $this->showError('404', 'error', 'unauthorized access');
10796: 			}
10797: 
10798: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10799: 			{	
10800: 				$id_user = $_SESSION[UID];
10801: 			}
10802: 			else
10803: 			{	
10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10805: 				$s = "SELECT
10806: 						`id_user`
10807: 					FROM `users`
10808: 					WHERE `id_user` = '" . $id_user . "'
10809: 					LIMIT 1
10810: 					";
10811: 
10812: 				$q = query($s);
10813: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10814: 				$d = fetch_assoc($q);
10815: 				
10816: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10817: 			}
10818: 
10819: 			if (empty($id_favorite)) return $this->showError('404', 'error', 'empty favorite');
10820: 			$favorite = explode(',', $id_favorite);
10821: 			$favorite_count = count($favorite );
10822: 			if (in_array($id_user, $favorite))
10823: 			{
10824: 				return $this->showError('404', 'error', 'trying to make user favorite for yourself');
10825: 			}
10826: 
10827: 			$s = "SELECT 
10828: 					COUNT(`users`.`id_user`) as f_in_u_count,
10829: 					GROUP_CONCAT(`users`.`id_user` SEPARATOR ',') as  f_in_u,
10830: 					GROUP_CONCAT(IF(`users`.`deleted`,`users`.`id_user`,NULL) SEPARATOR ',') as deleted,
10831: 					GROUP_CONCAT(uf.`id_favorite` SEPARATOR ',') as f_in_f
10832: 				FROM `users`
10833: 				LEFT JOIN (
10834: 						SELECT
10835: 							`id_favorite`,
10836: 							`id_user`
10837: 						FROM
10838: 							`users_favorite`
10839: 						WHERE
10840: 							`id_user` = '" . $id_user . "'
10841: 					) uf ON uf.`id_favorite` = `users`.`id_user`
10842: 				WHERE
10843: 					`users`.`id_user` in (" .  $id_favorite . ")
10844: 				";
10845: 
10846: 			$q = query($s);
10847: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
10848: 			$d = fetch_assoc($q);
10849: 
10850: 			if ($d['f_in_u_count'] != $favorite_count) 
10851: 			{
10852: 				return $this->showError('404', 'error', 'only ' . $d['f_in_u_count'] . ' users:' . $d['f_in_u'] .' of ' . $favorite_count . ' found');
10853: 			}
10854: 			if (!empty($d['deleted'])) 
10855: 			{
10856: 				return $this->showError('404', 'error', 'users ' . $d['deleted'] . ' deleted');
10857: 			}
10858: 			if (!empty($d['f_in_f']))
10859: 			{
10860: 				return $this->showError('404', 'error', 'users ' . $d['f_in_f'] . ' are already favorite for ' . $id_user);
10861: 			}
10862: 
10863: 			$s = array();
10864: 			foreach ($favorite as $f)
10865: 			{
10866: 				$s[] = "('" . $f . "', '" . $id_user . "')";
10867: 			}
10868: 			$s = "INSERT INTO `users_favorite` (`id_favorite`,`id_user`) VALUES " . implode(",", $s);
10869: 
10870: 			$q = query($s);
10871: 			if ($q === false) return $this->showError('404', 'error', 'services insert failed');
10872: 
10873: 			return array(
10874: 				'code' 		=>	'200',
10875: 				'status' 	=>	'success'
10876: 			);
10877: 		}
```

### `addTaskLog` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `23045-23176`
```php
23045: 		public function addTaskLog($data = '')
23046: 		{
23047: 			if (empty($_SESSION[UID])) {
23048: 				return $this->showError('404', 'error', 'unauthorized access');
23049: 			}
23050: 
23051: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23052: 
23053: 			if (empty($data)) 
23054: 			{
23055: 				return $this->showError('404', 'error', 'empty data');
23056: 			}
23057: 
23058: 			$data = json_decode($data,true);
23059: 			
23060: 			if (empty($data) || !is_array($data)) 
23061: 			{
23062: 				return $this->showError('404', 'error', 'wrong data');
23063: 			}
23064: 
23065: 			$allowed_fields = array(
23066: 				'tl_id'	=>		array(
23067: 										'name'	=>	'id_task_list',
23068: 										'NULL'	=>	false
23069: 									),
23070: 				'account'	=>		array(
23071: 										'name'	=>	'id_account',
23072: 										'NULL'	=>	true
23073: 									),
23074: 				'task_comment'	=>		array(
23075: 										'name'	=>	'id_task_comment',
23076: 										'NULL'	=>	true
23077: 									),
23078: 				'task_action_function'	=>	array(
23079: 										'name'	=>	'id_task_action_function',
23080: 										'NULL'	=>	true
23081: 									),
23082: 				'task_action'	=>		array(
23083: 										'name'	=>	'id_task_action',
23084: 										'NULL'	=>	true
23085: 									),
23086: 				'task_action_control'	=>		array(
23087: 										'name'	=>	'id_task_action_control',
23088: 										'NULL'	=>	true
23089: 									),
23090: 				'custom_comment'	=>		array(
23091: 										'name'	=>	'comment',
23092: 										'NULL'	=>	true
23093: 									),
23094: 				'custom_account'	=>	array(
23095: 										'name'	=>	'account',
23096: 										'NULL'	=>	true
23097: 									),
23098: 				'send_account'	=>		array(
23099: 										'name'	=>	'send_account',
23100: 										'NULL'	=>	true
23101: 									),
23102: 				'json'	=>		array(
23103: 										'name'	=>	'json',
23104: 										'NULL'	=>	false
23105: 									)
23106: 			);
23107: 
23108: 			$forbidden_fields = array();
23109: 			$affected_fields = array();
23110: 			$filtered_data = array();
23111: 			$add_table_list = array();
23112: 			foreach ($data as $key => $value)
23113: 			{
23114: 				if (isset($allowed_fields[$key]))
23115: 				{
23116: 					$affected_fields[] = $key;
23117: 					if (!empty($allowed_fields[$key]['format']))
23118: 					{
23119: 						$value = $allowed_fields[$key]['format']($value);
23120: 					}
23121: 					if (empty($value['error']))
23122: 					{								
23123: 						$name = $allowed_fields[$key]['name'];
23124: 						$null_on = $allowed_fields[$key]['NULL'];
23125: 						if ($null_on === true || $value !== NULL)
23126: 						{
23127: 							$filtered_data[$name] = $value;
23128: 						}
23129: 						else
23130: 						{
23131: 							return $this->showError('404', 'error', "$key: null value");
23132: 						}
23133: 					}
23134: 					else
23135: 					{
23136: 						return $this->showError('404', 'error', "$key: {$value['error']}");
23137: 					}
23138: 				}
23139: 				else
23140: 				{
23141: 					$forbidden_fields[] = $key;
23142: 				}
23143: 			}
23144: 
23145: 			if (empty($filtered_data)) return $this->showError('404', 'error', 'allowed data not found');
23146: 
23147: 			if (empty($filtered_data['id_task_list'])) return $this->showError('404', 'error', 'empty tl_id');
23148: 
23149: 			$s = array();
23150: 			foreach ($filtered_data as $key => $value)
23151: 			{
23152: 				$s[] = "`$key` = " 
23153: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
23154: 			}
23155: 
23156: 			$s = "INSERT INTO `task_list_log`
23157: 				SET 
23158: 					" . implode(",\n", $s) .",
23159: 				  `last_edit_timestamp` = now(),
23160: 				  `create_timestamp` = now(),
23161: 				  `last_edit_datetime` = now(),
23162: 				  `create_datetime` = now()
23163: 				";
23164: 
23165: 			$q = query($s);
23166: 			if ($q === false) return $this->showError('404', 'error', 'insert failed');
23167: 
23168: 			$out = array('affected_fields' 	=> 	$affected_fields);
23169: 			if (!empty($forbidden_fields)) $out['forbidden_fields'] = $forbidden_fields;
23170: 
23171: 			return array(
23172: 				'code' 		=>	'200',
23173: 				'status' 	=>	'success',		
23174: 				'data' 		=>	$out
23175: 			);
23176: 		}
```

### `cancelDeal` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `26582-26694`
```php
26582: 		public function cancelDeal($id_deal = '')
26583: 		{		
26584: 			if (empty($_SESSION[UID])) {
26585: 				return $this->showError('404', 'error', 'unauthorized access');
26586: 			}
26587: 
26588: 			$q = query("BEGIN");
26589: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
26590: 
26591: 			$s = "SELECT			
26592: 				  `id_deal`,
26593: 				  `customer`,
26594: 				  `performer`,
26595: 				  `sum`,
26596: 				  `currency`,
26597: 				  `commission`,
26598: 				  `performer_to_customer`,
26599: 				  `id_deal_status`,
26600: 				  `customer_account`
26601: 				FROM `deal`
26602: 				WHERE	
26603: 					`id_deal` = '$id_deal'
26604: 				LIMIT 1
26605: 				FOR UPDATE
26606: 				";
26607: 
26608: 			$q = query($s);
26609: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
26610: 			$d = fetch_assoc($q);
26611: 			if (empty($d['id_deal'])) return $this->showError('404', 'error', 'deal not found');
26612: 			if ($d['id_deal_status'] != 3 && $d['id_deal_status'] != 1) return $this->showError('404', 'error', 'wrong d_status');
26613: 			if ($this->id_role != 4 && ($d['customer'] != $_SESSION[UID] || $d['performer'] != $_SESSION[UID])) return $this->showError('404', 'error', 'user not customer or performer');
26614: 
26615: 			if ($d['id_deal_status'] == 3)
26616: 			{
26617: 				$s = "UPDATE `currency_account`
26618: 					SET
26619: 						`sum` = `sum` + '{$d['sum']}',
26620: 						`reserved` = `reserved` - '{$d['sum']}',
26621: 						`last_edit_user` = '{$_SESSION[UID]}',
26622: 						`last_edit_int_timestamp` =  '" . time() . "'
26623: 					WHERE
26624: 						`id_currency_account` = '{$d['customer_account']}'
26625: 					";
26626: 
26627: 				$q = query($s);
26628: 
26629: 				$update = affected_rows();
26630: 				if ($update === -1) 
26631: 				{
26632: 					return $this->showError('400','error','account update failed'. error_db());
26633: 				}
26634: 				elseif ($update  == 0) 
26635: 				{
26636: 					$warning[] = 'modified data not found';
26637: 				}
26638: 
26639: 				$now_time = time();
26640: 				$s = "INSERT INTO `transaction`
26641: 					SET 
26642: 					  `sum` = '{$d['sum']}',
26643: 					  `currency` = '{$d['currency']}',
26644: 					  `total_sum` = '{$d['sum']}',
26645: 					  `commission` =  '0',
26646: 					  `id_transaction_type` = '4',
26647: 					  `create_user` = '{$_SESSION[UID]}',
26648: 					  `last_edit_user` = '{$_SESSION[UID]}',
26649: 					  `last_edit_int_timestamp` = '$now_time',
26650: 					  `create_int_timestamp` = '$now_time',
26651: 					  `id_transaction_status` = '2',
26652: 					  `id_deal` = '$id_deal',
26653: 					  `to` = '{$d['customer_account']}'
26654: 					";
26655: 
26656: 				$q = query($s);
26657: 				if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
26658: 
26659: 				$trn_id = insert_id();
26660: 				$out['trn_id'] = $trn_id;	
26661: 			}
26662: 
26663: 			$s = "UPDATE `deal`
26664: 				SET
26665: 					`id_deal_status` = '4',
26666: 					`last_edit_user` = '{$_SESSION[UID]}',
26667: 					`last_edit_int_timestamp` =  '" . time() . "'
26668: 				WHERE
26669: 					`id_deal` = '$id_deal'
26670: 				";
26671: 
26672: 			$q = query($s);
26673: 
26674: 			$update = affected_rows();
26675: 			if ($update === -1) 
26676: 			{
26677: 				return $this->showError('400','error','deal update failed'. error_db());
26678: 			}
26679: 			elseif ($update == 0) 
26680: 			{
26681: 				$warning[] = 'deal modified data not found';
26682: 			}
26683: 
26684: 			$q = query("COMMIT");
26685: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
26686: 
26687: 			if (!empty($warning)) $out['warning'] = $warning;
26688: 
26689: 			return array(
26690: 				'code' 		=>	'200',
26691: 				'status' 	=>	'success',		
26692: 				'data' 		=>	isset($out) ? $out : array()
26693: 			);
26694: 		}
```

### `cancelOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 4=Administrator, 5=Agent`
- source range: `7050-7394`
```php
7050: 		public function cancelOrder($id_order = "", $forced = 0, $cancel_reason = "", $cancel_states = "", $site_cancel_states = array())
7051: 		{
7052: 			if (empty($_SESSION[UID])) {
7053: 				return $this->showError('404', 'error', 'unauthorized access');
7054: 			}
7055: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5 && $this->id_role != 4)
7056: 			{
7057: 				return $this->showError('404', 'error', 'wrong user role');
7058: 			}
7059: 			if (!empty($cancel_states))
7060: 			{
7061: 				$s_cancel_states = array();
7062: 				$forbidden_cancel_states = array();
7063: 				foreach (explode(',', $cancel_states) as $c_s)
7064: 				{
7065: 					$c_s = trim($c_s);
7066: 					if (!empty($site_cancel_states[$c_s]['user_roles']) && in_array($this->id_role,$site_cancel_states[$c_s]['user_roles']))
7067: 					{
7068: 						$s_cancel_states[] = "('" . $id_order . "', '" . $c_s . "', '" . $_SESSION[UID] . "')";
7069: 					}
7070: 					else
7071: 					{
7072: 						$forbidden_cancel_states[] = $c_s;
7073: 					}
7074: 				}
7075: 				if (!empty($forbidden_cancel_states))
7076: 				{
7077: 					return $this->showError('404', 'error', 'wrong booking_cancel_states: ' . implode(",", $forbidden_cancel_states));
7078: 				}
7079: 				$s_cancel_states = "INSERT INTO `order_cancel_items` (`id_order`,`id_order_cancel`,`id_user`) VALUES " . implode(",", $s_cancel_states);
7080: 			}
7081: 	
7082: 			if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
7083: 			{
7084: 				$s = "SELECT
7085: 						`id_order`,
7086: 						`client`,
7087: 						`id_order_status`,
7088: 						`pay_datetime`,
7089: 						`options`
7090: 					FROM `order` 		
7091: 					WHERE	
7092: 						`id_order` = '" . $id_order . "'
7093: 					LIMIT 1
7094: 					";
7095: 
7096: 				$q = query($s);
7097: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7098: 
7099: 				$d = fetch_assoc($q);
7100: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7101: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5 
7102: 					&& $d['id_order_status'] != 6) 
7103: 				{
7104: 					return $this->showError('404', 'error', 'wrong booking state');
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
7107: 				{
7108: 					return $this->showError('404', 'error', 'user is not author');
7109: 				}
7110: 
7111: 				if (DEFAULT_PROFILE == 'stadium')
7112: 				{
7113: 					if ($d['pay_datetime'] !== '0000-00-00 00:00:00') return $this->showError('404', 'error', 'order has already been paid');
7114: 					$d['options'] = json_decode($d['options'],true);
7115: 					$d_options = $d['options'];
7116: 					if (empty($d['options']['tickets']['payment'])) $d['options']['tickets']['payment'] = '';
7117: 					if ($d['options']['tickets']['payment'] == 'succeeded')
7118: 					{
7119: 						return $this->showError('404', 'error', 'order with succeeded status');
7120: 					}
7121: 
7122: 					if ($d['options']['tickets']['payment'] !== 'failed')
7123: 					{
7124: 						$api_use = true;
7125: 						$order = $id_order;
7126: 						$status = 'failed';
7127: 						$cancel_status = cancel_stripe_link($id_order);
7128: 						if (!empty($cancel_status['error'])) 
7129: 						{
7130: 							if (empty($cancel_status['error'][1]))
7131: 							{
7132: 								if (empty($forced)) return $this->showError('404', 'error', $cancel_status['error']);
7133: 							}
7134: 							elseif ($cancel_status['error'][1] === "Only Checkout Sessions with a status in [\"open\"] can be expired. This Checkout Session has a status of `complete`.")
7135: 							{
7136: 								$status = 'succeeded';
7137: 							}
7138: 							elseif ($cancel_status['error'][1] !== "Only Checkout Sessions with a status in [\"open\"] can be expired. This Checkout Session has a status of `expired`.")
7139: 							{
7140: 								if (empty($forced)) return $this->showError('404', 'error', $cancel_status['error']);
7141: 							}
7142: 						}
7143: 						$res = require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'controllers/c_stripe.php');
7144: 						if (!empty($res['error'])) return $this->showError('404', 'error', $res['error']);				
7145: 					}
7146: 				}
7147: 				if (empty($status) || $status != 'succeeded')
7148: 				{
7149: 					$q = query("BEGIN");
7150: 					if ($q === false) return $this->showError('404', 'error', 'begin query failed');
7151: 
7152: 					if (!empty($s_cancel_states))
7153: 					{
7154: 						$q = query($s_cancel_states);
7155: 						if ($q === false) return $this->showError('404', 'error', 'booking_cancel_states insert failed');
7156: 					}
7157: 
7158: 					$s = "UPDATE `order`
7159: 						SET 
7160: 							`id_order_status` = '3',
7161: 							`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7162: 							`cancel_datetime` = now(),
7163: 							`last_edit_datetime` = now(),
7164: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7165: 						WHERE
7166: 							`id_order` = '" . $id_order . "' AND `id_order_status`in (1,2,5,6)
7167: 						";
7168: 
7169: 					query($s);
7170: 			
7171: 					global $link;
7172: 					if (mysqli_affected_rows($link) === -1) 
7173: 					{
7174: 						return $this->showError('404', 'error', 'booking update failed');
7175: 					}
7176: 					elseif (mysqli_affected_rows($link) === 0) 
7177: 					{
7178: 						if (empty($status)) return $this->showError('404', 'error', 'booking modified data not found');
7179: 					}
7180: 
7181: 					$q = query("COMMIT");
7182: 					if ($q === false) return $this->showError('404', 'error', 'commit query failed');
7183: 				}
7184: 			}
7185: 			else
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
7209: 				$q = query($s);
7210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7211: 
7212: 				$d = fetch_assoc($q);
7213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7214: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2
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
7227: 					if ($q === false) return $this->showError('404', 'error', 'begin query failed');
7228: 					
7229: 					if (!empty($s_cancel_states))
7230: 					{
7231: 						$q = query($s_cancel_states);
7232: 						if ($q === false) return $this->showError('404', 'error', 'booking_cancel_states insert failed');
7233: 					}
7234: 
7235: 					$s = "UPDATE `order`,`order_driver`
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
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
7253: 					if (mysqli_affected_rows($link) === -1) 
7254: 					{
7255: 						return $this->showError('404', 'error', 'driver update failed');
7256: 					}
7257: 					elseif (mysqli_affected_rows($link) === 0) 
7258: 					{
7259: 						return $this->showError('404', 'error', 'driver modified data not found');
7260: 					}
7261: 
7262: 					$q = query("COMMIT");
7263: 					if ($q === false) return $this->showError('404', 'error', 'commit query failed');
7264: 				}
7265: 				elseif ($d['id_order_driver_status'] == 2)
7266: 				{
7267: 					return $this->showError('404', 'error', 'canceled performer');
7268: 				}
7269: 				elseif ($d['id_order_driver_status'] == 3 || $d['id_order_driver_status'] == 4)
7270: 				{
```

### `checkTicket` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator, 6=Usher`
- source range: `24225-24294`
```php
24225: 		public function checkTicket($code = '')
24226: 		{
24227: 			if (empty($_SESSION[UID])) {
24228: 				return $this->showError('404', 'error', 'unauthorized access');
24229: 			}
24230: 			if (empty($code)) 
24231: 			{
24232: 				return $this->showError('404', 'error', 'empty code');
24233: 			}
24234: 			if ($this->id_role != 4 && $this->id_role != 6) 
24235: 			{
24236: 				return $this->showError('404', 'error', 'wrong role');
24237: 			}
24238: 
24239: 			$sql_field = '`ticket`.`id_schedule` as sc_id,
24240: 					`ticket`.`id_order` as b_id,
24241: 					`ticket`.`id_trip_seat` as ti_t_id,';
24242: 			$sql_join = '';
24243: 			$sql_where = "`ticket`.`code` = '$code'";
24244: 
24245: 			if ($this->id_role == 6)
24246: 			{
24247: 				$sql_field = '';
24248: 				$sql_join = "JOIN (
24249: 					SELECT
24250: 							`id_schedule`
24251: 						FROM
24252: 							`users`
24253: 						WHERE
24254: 							`id_user` = '" . $_SESSION[UID] . "'
24255: 						LIMIT 1
24256: 					) u";
24257: 				$sql_where .= " AND u.`id_schedule` = `ticket`.`id_schedule`";
24258: 			
24259: 			}
24260: 
24261: 			$s = "SELECT
24262: 					$sql_field
24263: 					`ticket`.`id_seat` as seat,
24264: 					`ticket`.`id_trip` as t_id,
24265: 					`ticket`.`pass`,
24266: 					`ticket`.`pass_datetime`,
24267: 					`ticket`.`out_datetime`,
24268: 					`ticket`.`status`,
24269: 					`order`.`pay_datetime` as b_payment_datetime,
24270: 					`order`.`id_order_status` as b_state
24271: 				FROM `ticket`
24272: 				$sql_join
24273: 				LEFT JOIN `order` USING (`id_order`)
24274: 				WHERE 
24275: 					 $sql_where
24276: 				LIMIT 1
24277: 				";
24278: 
24279: 			$q = query($s);
24280: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
24281: 	
24282: 			$d = fetch_assoc($q);
24283: 			if (empty($d['seat'])) return $this->showError('404', 'error', 'ticket not found');
24284: 			
24285: 			add_time_zone($d['pass_datetime']);
24286: 			add_time_zone($d['out_datetime']);
24287: 			add_time_zone($d['b_payment_datetime']);
24288: 
24289: 			return array(
24290: 				'code' 		=>	'200',
24291: 				'status' 	=>	'success',		
24292: 				'data' 		=>	$d
24293: 			);
24294: 		}
```

### `completeDeal` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `26406-26580`
```php
26406: 		public function completeDeal($id_deal = '')
26407: 		{		
26408: 			if (empty($_SESSION[UID])) {
26409: 				return $this->showError('404', 'error', 'unauthorized access');
26410: 			}
26411: 
26412: 			$q = query("BEGIN");
26413: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
26414: 
26415: 			$s = "SELECT			
26416: 				  `id_deal`,
26417: 				  `customer`,
26418: 				  `performer`,
26419: 				  `sum`,
26420: 				  `currency`,
26421: 				  `commission`,
26422: 				  `performer_to_customer`,
26423: 				  `id_deal_status`,
26424: 				  `customer_account`
26425: 				FROM `deal`
26426: 				WHERE	
26427: 					`id_deal` = '$id_deal'
26428: 				LIMIT 1
26429: 				FOR UPDATE
26430: 				";
26431: 
26432: 			$q = query($s);
26433: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
26434: 			$d = fetch_assoc($q);
26435: 			if (empty($d['id_deal'])) return $this->showError('404', 'error', 'deal not found');
26436: 			if ($d['id_deal_status'] != 3) return $this->showError('404', 'error', 'wrong d_status');
26437: 			if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
26438: 
26439: 			$s = "SELECT			
26440: 				  `id_currency_account`,
26441: 				  `id_user`,
26442: 				  `id_currency_account_status`,
26443: 				  `sum`
26444: 				FROM `currency_account`
26445: 				WHERE	
26446: 					`id_user` = '{$d['performer']}' AND `currency` = '{$d['currency']}'
26447: 				LIMIT 1
26448: 				FOR UPDATE
26449: 				";
26450: 
26451: 			$q = query($s);
26452: 			if ($q === false) return $this->showError('404', 'error', 'to select failed');
26453: 			$d_ca = fetch_assoc($q);
26454: 			if (empty($d_ca['id_currency_account']))
26455: 			{
26456: 					$now_time = time();
26457: 					$s = "INSERT INTO `currency_account`
26458: 						SET 
26459: 						  `id_user` = '{$d['performer']}',
26460: 						  `sum`= 0,
26461: 						  `currency` = '{$d['currency']}',
26462: 						  `reserved` = 0,
26463: 						  `create_user` = '{$_SESSION[UID]}',
26464: 						  `last_edit_user` = '{$_SESSION[UID]}',
26465: 						  `last_edit_int_timestamp` = '$now_time',
26466: 						  `create_int_timestamp` = '$now_time'
26467: 						";
26468: 
26469: 					$q = query($s);
26470: 					if ($q === false) return $this->showError('404', 'error', 'insert failed');
26471: 					$a_id = insert_id();
26472: 					$out['a_id'] = $a_id;
26473: 			}
26474: 			else
26475: 			{
26476: 				if ($d_ca['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong to account status');
26477: 				$a_id = $d_ca['id_currency_account'];
26478: 			}
26479: 
26480: 			$s = "UPDATE `currency_account`
26481: 				SET
26482: 					`reserved` = `reserved` - '{$d['sum']}',
26483: 					`last_edit_user` = '{$_SESSION[UID]}',
26484: 					`last_edit_int_timestamp` =  '" . time() . "'
26485: 				WHERE
26486: 					`id_currency_account` = '{$d['customer_account']}'
26487: 				";
26488: 
26489: 			$q = query($s);
26490: 
26491: 			$update = affected_rows();
26492: 			if ($update === -1) 
26493: 			{
26494: 				return $this->showError('400','error','account update failed'. error_db());
26495: 			}
26496: 			elseif ($update  == 0) 
26497: 			{
26498: 				$warning[] = 'modified data not found';
26499: 			}
26500: 
26501: 			$total_sum = ((float)$d['sum']) * (1  - ($this->constant['commission_deal']) / 100);
26502: 
26503: 			$s = "UPDATE `currency_account`
26504: 				SET
26505: 					`sum` = `sum` + '$total_sum',
26506: 					`last_edit_user` = '{$_SESSION[UID]}',
26507: 					`last_edit_int_timestamp` =  '" . time() . "'
26508: 				WHERE
26509: 					`id_currency_account` = '$a_id'
26510: 				";
26511: 
26512: 			$q = query($s);
26513: 
26514: 			$update = affected_rows();
26515: 			if ($update === -1) 
26516: 			{
26517: 				return $this->showError('400','error','currency account update failed'. error_db());
26518: 			}
26519: 			elseif ($update == 0) 
26520: 			{
26521: 				$warning[] = 'modified data not found for currency account';
26522: 			}
26523: 
26524: 			$now_time = time();
26525: 			$s = "INSERT INTO `transaction`
26526: 				SET 
26527: 				  `sum` = '{$d['sum']}',
26528: 				  `currency` = '{$d['currency']}',
26529: 				  `total_sum` = '$total_sum',
26530: 				  `commission` =  '{$this->constant['commission_deal']}',
26531: 				  `id_transaction_type` = '8',
26532: 				  `create_user` = '{$_SESSION[UID]}',
26533: 				  `last_edit_user` = '{$_SESSION[UID]}',
26534: 				  `last_edit_int_timestamp` = '$now_time',
26535: 				  `create_int_timestamp` = '$now_time',
26536: 				  `id_transaction_status` = '2',
26537: 				  `id_deal` = '$id_deal',
26538: 				  `from` = '{$d['customer_account']}',
26539: 				  `to` = '$a_id'
26540: 				";
26541: 
26542: 			$q = query($s);
26543: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
26544: 
26545: 			$trn_id = insert_id();
26546: 			$out['trn_id'] = $trn_id;
26547: 
26548: 			$s = "UPDATE `deal`
26549: 				SET
26550: 					`id_deal_status` = '5',
26551: 					`performer_account` = '$a_id',
26552: 					`last_edit_user` = '{$_SESSION[UID]}',
26553: 					`last_edit_int_timestamp` =  '" . time() . "'
26554: 				WHERE
26555: 					`id_deal` = '$id_deal'
26556: 				";
26557: 
26558: 			$q = query($s);
26559: 
26560: 			$update = affected_rows();
26561: 			if ($update === -1) 
26562: 			{
26563: 				return $this->showError('400','error','deal update failed'. error_db());
26564: 			}
26565: 			elseif ($update == 0) 
26566: 			{
26567: 				$warning[] = 'deal modified data not found';
26568: 			}
26569: 
26570: 			$q = query("COMMIT");
26571: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
26572: 
26573: 			if (!empty($warning)) $out['warning'] = $warning;
26574: 
26575: 			return array(
26576: 				'code' 		=>	'200',
26577: 				'status' 	=>	'success',		
26578: 				'data' 		=>	$out
26579: 			);
26580: 		}
```

### `completeOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `6896-7048`
```php
6896: 		public function completeOrder($id_order = "")
6897: 		{
6898: 			if (empty($_SESSION[UID])) {
6899: 				return $this->showError('404', 'error', 'unauthorized access');
6900: 			}
6901: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6902: 			{
6903: 				return $this->showError('404', 'error', 'wrong user role');
6904: 			}
6905: 
6906: 			if ($this->id_role == 1 || $this->id_role == 5)
6907: 			{
6908: 				$s = "SELECT
6909: 						`id_order`,
6910: 						`client`,
6911: 						`id_order_status`
6912: 					FROM `order` 		
6913: 					WHERE	
6914: 						`id_order` = '" . $id_order . "'
6915: 					LIMIT 1
6916: 					";
6917: 
6918: 				$q = query($s);
6919: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6920: 
6921: 				$d = fetch_assoc($q);
6922: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6923: 				if ($d['id_order_status'] != 2) return $this->showError('404', 'error', 'wrong booking state');
6924: 				if ($d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not author');
6925: 
6926: 				$s = "UPDATE `order`
6927: 					SET 
6928: 						`id_order_status` = '4',
6929: 						`complete_datetime` = now(),
6930: 						`last_edit_datetime` = now(),
6931: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
6932: 					WHERE
6933: 						`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
6934: 					";
6935: 
6936: 				query($s);
6937: 		
6938: 				global $link;
6939: 				if (mysqli_affected_rows($link) === -1) 
6940: 				{
6941: 					return $this->showError('404', 'error', 'database update failed');
6942: 				}
6943: 				elseif (mysqli_affected_rows($link) === 0) 
6944: 				{
6945: 
6946: 					return $this->showError('404', 'error', 'modified data not found');
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
6961: 					FROM `order` 
6962: 
6963: 					LEFT JOIN `order_driver` USING (`id_order`)				
6964: 					WHERE	
6965: 						`order`.`id_order` = '" . $id_order . "'
6966: 					LIMIT 1
6967: 					";
6968: 
6969: 				$q = query($s);
6970: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6971: 
6972: 				$d = fetch_assoc($q);
6973: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6974: 				if ($d['id_order_status'] != 2 && $d['id_order_status'] != 4)
6975: 				{
6976: 					return $this->showError('404', 'error', 'wrong booking state');
6977: 				}
6978: 				if (empty($d['u_id'])) 
6979: 				{
6980: 					return $this->showError('404', 'error', 'user is not performer');
6981: 				}			
6982: 				if ($d['c_state'] == 1) 
6983: 				{
6984: 					return $this->showError('404', 'error', 'not appointed performer');
6985: 
6986: 				}
6987: 				elseif ($d['c_state'] == 2)
6988: 				{
6989: 					return $this->showError('404', 'error', 'canceled performer');
6990: 				}
6991: 				global $link;
6992: 				if ($d['c_state'] != 6)
6993: 				{
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
7006: 						";
7007: 
7008: 					query($s);
7009: 					
7010: 					if (mysqli_affected_rows($link) === -1) 
7011: 					{
7012: 						return $this->showError('404', 'error', 'driver update failed');
7013: 					}
7014: 					elseif (mysqli_affected_rows($link) === 0) 
7015: 					{
7016: 						return $this->showError('404', 'error', 'driver modified data not found');
7017: 					}
7018: 				}
7019: 				if ($d['id_order_status'] == 2 && empty($d['incomplete_count']))
7020: 				{
7021: 					$s = "UPDATE `order`
7022: 						SET 
7023: 							`id_order_status` = '4',
7024: 							`complete_datetime` = now(),
7025: 							`last_edit_datetime` = now(),
7026: 							`last_edit_user` = '" .  $_SESSION[UID] . "'
7027: 						WHERE
7028: 							`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
7029: 						";
7030: 
7031: 					query($s);
7032: 			
7033: 					if (mysqli_affected_rows($link) === -1) 
7034: 					{
7035: 						return $this->showError('404', 'error', 'booking update failed');
7036: 					}
7037: 					elseif (mysqli_affected_rows($link) === 0) 
7038: 					{
7039: 						return $this->showError('404', 'error', 'booking modified data not found');
7040: 					}
7041: 				}			
7042: 			}
7043: 
7044: 			return array(
7045: 				'code' 		=>	'200',
7046: 				'status' 	=>	'success'
7047: 			);	
7048: 		}
```

### `confirmDeal` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `26752-26891`
```php
26752: 		public function confirmDeal($id_deal = '')
26753: 		{		
26754: 			if (empty($_SESSION[UID])) {
26755: 				return $this->showError('404', 'error', 'unauthorized access');
26756: 			}
26757: 
26758: 			$q = query("BEGIN");
26759: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
26760: 
26761: 			$s = "SELECT			
26762: 				  `id_deal`,
26763: 				  `customer`,
26764: 				  `performer`,
26765: 				  `sum`,
26766: 				  `currency`,
26767: 				  `commission`,
26768: 				  `performer_to_customer`,
26769: 				  `id_deal_status`
26770: 				FROM `deal`
26771: 				WHERE	
26772: 					`id_deal` = '$id_deal'
26773: 				LIMIT 1
26774: 				FOR UPDATE
26775: 				";
26776: 
26777: 			$q = query($s);
26778: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
26779: 			$d = fetch_assoc($q);
26780: 			if (empty($d['id_deal'])) return $this->showError('404', 'error', 'deal not found');
26781: 			if ($d['id_deal_status'] != 1) return $this->showError('404', 'error', 'wrong d_status');
26782: 			if (empty($d['performer_to_customer']))
26783: 			{
26784: 				if ($this->id_role != 4 && $d['performer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not performer');
26785: 			}
26786: 			else
26787: 			{
26788: 				if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
26789: 			}
26790: 			
26791: 			$customer_account_sql = '`customer_account`';
26792: 			if (!empty($d['performer_to_customer']))
26793: 			{
26794: 				$s = "SELECT			
26795: 					  `id_currency_account`,
26796: 					  `id_user`,
26797: 					  `id_currency_account_status`,
26798: 					  `sum`
26799: 					FROM `currency_account`
26800: 					WHERE	
26801: 						`id_user` = '{$d['customer']}' AND `currency` = '{$d['currency']}'
26802: 					LIMIT 1
26803: 					FOR UPDATE
26804: 					";
26805: 
26806: 				$q = query($s);
26807: 				if ($q === false) return $this->showError('404', 'error', 'from select failed');
26808: 				$d_ca = fetch_assoc($q);
26809: 				if (empty($d_ca['id_currency_account'])) return $this->showError('404', 'error', 'account not found');
26810: 				if ($d_ca['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong from account status');
26811: 				if ((int)$d_ca['sum'] < (int)$d['sum']) return $this->showError('404', 'error', 'too much sum');
26812: 				
26813: 				$s = "UPDATE `currency_account`
26814: 					SET
26815: 						`sum` = `sum` - '{$d['sum']}',
26816: 						`reserved` = `reserved` + '{$d['sum']}',
26817: 						`last_edit_user` = '{$_SESSION[UID]}',
26818: 						`last_edit_int_timestamp` =  '" . time() . "'
26819: 					WHERE
26820: 						`id_currency_account` = '{$d_ca['id_currency_account']}'
26821: 					";
26822: 
26823: 				$q = query($s);
26824: 
26825: 				$update = affected_rows();
26826: 				if ($update === -1) 
26827: 				{
26828: 					return $this->showError('400','error','account update failed'. error_db());
26829: 				}
26830: 				elseif ($update == 0) 
26831: 				{
26832: 					$warning[] = 'modified data not found';
26833: 				}
26834: 
26835: 				$now_time = time();
26836: 				$s = "INSERT INTO `transaction`
26837: 					SET 
26838: 					  `sum` = '{$d['sum']}',
26839: 					  `currency` = '{$d['currency']}',
26840: 					  `total_sum` = '{$d['sum']}',
26841: 					  `commission` = '0',
26842: 					  `id_transaction_type` = '3',
26843: 					  `create_user` = '{$_SESSION[UID]}',
26844: 					  `last_edit_user` = '{$_SESSION[UID]}',
26845: 					  `last_edit_int_timestamp` = '$now_time',
26846: 					  `create_int_timestamp` = '$now_time',
26847: 					  `id_transaction_status` = '2',
26848: 					  `id_deal` = '$id_deal',
26849: 					  `from` = '{$d_ca['id_currency_account']}'
26850: 					";
26851: 
26852: 				$q = query($s);
26853: 				if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
26854: 
26855: 				$trn_id = insert_id();
26856: 				$out['trn_id'] = $trn_id;
26857: 				$customer_account_sql = "'{$d_ca['id_currency_account']}'";
26858: 			}
26859: 
26860: 			$s = "UPDATE `deal`
26861: 				SET
26862: 					`id_deal_status` = '3',
26863: 					`customer_account` = $customer_account_sql,
26864: 					`last_edit_user` = '{$_SESSION[UID]}',
26865: 					`last_edit_int_timestamp` =  '" . time() . "'
26866: 				WHERE
26867: 					`id_deal` = '$id_deal'
26868: 				";
26869: 
26870: 			$q = query($s);
26871: 
26872: 			$update = affected_rows();
26873: 			if ($update === -1) 
26874: 			{
26875: 				return $this->showError('400','error','deal update failed'. error_db());
26876: 			}
26877: 			elseif ($update == 0) 
26878: 			{
26879: 				$warning[] = 'deal modified data not found';
26880: 			}
26881: 
26882: 			$q = query("COMMIT");
26883: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
26884: 
26885: 			if (!empty($warning)) $out['warning'] = $warning;
26886: 			return array(
26887: 				'code' 		=>	'200',
26888: 				'status' 	=>	'success',		
26889: 				'data' 		=>	isset($out) ? $out : array()
26890: 			);
26891: 		}
```

### `confirmOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 5=Agent`
- source range: `8181-8246`
```php
8181: 		public function confirmOrder($id_order = "", $estimated_waiting_time = NULL)
8182: 		{	
8183: 			if (empty($_SESSION[UID])) {
8184: 				return $this->showError('404', 'error', 'unauthorized access');
8185: 			}
8186: 			if ($this->id_role != 1 && $this->id_role != 5)
8187: 			{
8188: 				return $this->showError('404', 'error', 'wrong user role');
8189: 			}
8190: 
8191: 			$s = "SELECT
8192: 					`id_order`,
8193: 					`client`,
8194: 					`id_order_status`,
8195: 					`is_confirmed`
8196: 				FROM `order` 		
8197: 				WHERE	
8198: 					`id_order` = '" . $id_order . "'
8199: 				LIMIT 1
8200: 				";
8201: 
8202: 			$q = query($s);
8203: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8204: 
8205: 			$d = fetch_assoc($q);
8206: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8207: 			if ($d['client'] != $_SESSION[UID]) 
8208: 			{
8209: 				return $this->showError('404', 'error', 'user is not author');
8210: 
8211: 			}
8212: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5
8213: 				&& $d['id_order_status'] != 6)
8214: 			{
8215: 				return $this->showError('404', 'error', 'wrong booking state');
8216: 			}		
8217: 			if (!empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking already confirmed');
8218: 
8219: 			$s = "UPDATE `order`
8220: 				SET 
8221: 					`is_confirmed` = '1',
8222: 					`confirm_datetime` = now() " . ($estimated_waiting_time !== NULL ? ",
8223: 					`estimated_waiting_datetime` = IF(`datetime_start_plan` = 0,`create_datetime`,`datetime_start_plan`) + INTERVAL '" . $estimated_waiting_time . "' SECOND" : '') . ",
8224: 					`last_edit_datetime` = now(),
8225: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
8226: 				WHERE
8227: 					`id_order` = '" . $id_order . "' AND `is_confirmed` = '0'
8228: 				";
8229: 
8230: 			query($s);
8231: 	
8232: 			global $link;
8233: 			if (mysqli_affected_rows($link) === -1) 
8234: 			{
8235: 				return $this->showError('404', 'error', 'database update failed');
8236: 			}
8237: 			elseif (mysqli_affected_rows($link) === 0) 
8238: 			{
8239: 				return $this->showError('404', 'error', 'modified data not found');
8240: 			}
8241: 
8242: 			return array(
8243: 				'code' 		=>	'200',
8244: 				'status' 	=>	'success'
8245: 			);
8246: 		}
```

### `controlCar` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `2066-2340`
```php
2066: 		public function controlCar($data = "", $id_car = "", $id_user = "", $langs = array(), $order_location = array(), $currency = array(), $country = array(),$region = array(),$city = array())
2067: 		{
2068: 			if (empty($_SESSION[UID])) {
2069: 				return $this->showError('404', 'error', 'unauthorized access');
2070: 			}	
2071: 			if (empty($data)) 
2072: 			{
2073: 				return $this->showError('404', 'error', 'empty data');
2074: 			}
2075: 
2076: 			$data = json_decode($data,true);
2077: 			
2078: 			if (empty($data) || !is_array($data)) 
2079: 			{
2080: 				return $this->showError('404', 'error', 'wrong data');
2081: 			}
2082: 
2083: 			$allowed_fields = array(							
2084: 									'cm_id'					=>		'id_car_model',
2085: 									'seats'					=>		'seats',
2086: 									'registration_plate'	=>		'license_plate',
2087: 									'color'					=>		'id_car_color',
2088: 									'photo'					=>		'photo_link',
2089: 									'details'				=>		'json',
2090: 									'cc_id'					=>		'id_car_class',
2091: 									'licenses'				=>		empty($id_car) ? '' : 'licenses'
2092: 			);
2093: 
2094: 			$forbidden_fields = array();
2095: 			$affected_fields = array();
2096: 
2097: 			$affected_keys = array();
2098: 			$filtered_data = array();
2099: 			$reassign = array();		
2100: 			foreach ($data as $key => $value)
2101: 			{
2102: 				if (!empty($allowed_fields[$key]))
2103: 				{
2104: 					if (is_string($value)) $data[$key] = trim($value);
2105: 
2106: 					$affected_fields[] = $key;		
2107: 					$affected_keys[$key] = true;		
2108: 					$filtered_data[$allowed_fields[$key]] = $data[$key];
2109: 				}
2110: 				else
2111: 				{
2112: 					if ($key != "assign") $forbidden_fields[] = $key;
2113: 				}
2114: 			}
2115: 			
2116: 			if (!empty($affected_keys['photo']))
2117: 			{
2118: 				$filtered_data['photo_link'] = base64ImageToFile($filtered_data['photo_link']);
2119: 
2120: 				if ($filtered_data['photo_link']['error'] === false)
2121: 				{
2122: 					$filtered_data['photo_link'] = $filtered_data['photo_link']['image'];
2123: 				}
2124: 				else
2125: 				{
2126: 					return $this->showError('404', 'error', $filtered_data['photo_link']['error']);
2127: 				}
2128: 			}
2129: 
2130: 			if (!empty($affected_keys['details']))
2131: 			{
2132: 				$filtered_data['json'] = real_escape_string(json_encode($filtered_data['json']));
2133: 			}
2134: 
2135: 			if (empty($id_car))
2136: 			{
2137: 				if (empty($affected_fields)) return $this->showError('404', 'error', 'empty allowed data');
2138: 				if (empty($filtered_data['license_plate'])) 
2139: 				{
2140: 					return $this->showError('404', 'error', 'empty registration plate');
2141: 				}
2142: 
2143: 				if (empty($id_user) || $_SESSION[UID] == $id_user) 
2144: 				{
2145: 					if ($this->id_role == 2)
2146: 					{
2147: 						if (empty($_SESSION['id_verification_status']) 
2148: 							|| $_SESSION['id_verification_status'] == 1)
2149: 						{						
2150: 							$car = $this->createCar($filtered_data,$_SESSION[UID]);
2151: 							if (!empty($car['error'])) 
2152: 							{
2153: 								return $this->showError('404', 'error', $car['error']);
2154: 							}
2155: 							$user = $_SESSION[UID];
2156: 						}
2157: 						else
2158: 						{
2159: 							return $this->showError('404', 'error', 'wrong user check state');
2160: 						}
2161: 					}
2162: 					else
2163: 					{
2164: 						return $this->showError('404', 'error', 'wrong user role');
2165: 					}
2166: 				}
2167: 				else
2168: 				{
2169: 					if ($this->id_role != 4) 
2170: 					{
2171: 						return $this->showError('404', 'error', 'not enough rights');
2172: 					}
2173: 					
2174: 					$s = "SELECT 
2175: 							`id_role`,
2176: 							`id_user`
2177: 						FROM `users`
2178: 						WHERE 
2179: 							`id_user` = '" . $id_user . "'
2180: 						LIMIT 1
2181: 						";
2182: 
2183: 					$q = query($s);
2184: 					if ($q === false) return $this->showError('404', 'error', 'database select failed');
2185: 					$d = fetch_assoc($q);
2186: 
2187: 					if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');
2188: 					if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong role of user');
2189: 					
2190: 					$car = $this->createCar($filtered_data,$id_user);
2191: 					if (!empty($car['error'])) 
2192: 					{
2193: 						return $this->showError('404', 'error', $car['error']);
2194: 					}
2195: 					$user = $id_user;
2196: 				}
2197: 			}
2198: 			else
2199: 			{			
2200: 				if (!empty($id_user) && $this->id_role != 4) 
2201: 				{
2202: 					return $this->showError('404', 'error', 'not enough rights for assign');
2203: 				}
2204: 				
2205: 				$s = "SELECT
2206: 						`car`.`id_car`,
2207: 						`car`.`license_plate`,
2208: 						GROUP_CONCAT(`car_users`.`id_user` SEPARATOR ',') as u_id
2209: 					FROM `car`
2210: 					LEFT JOIN `car_users` USING (`id_car`)
2211: 					WHERE
2212: 						`car`.`id_car` = '" . $id_car . "'
2213: 					GROUP BY
2214: 						`car_users`.`id_car`
2215: 					";
2216: 
2217: 				$q = query($s);
2218: 				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
2219: 				$d = fetch_assoc($q);
2220: 
2221: 				if (empty($d['id_car'])) return $this->showError('404', 'error', 'car not found');
2222: 
2223: 				if (empty($id_user)) 
2224: 				{
2225: 					if ($this->id_role == 4 || (in_array($_SESSION[UID], explode(',',$d['u_id'])) 
2226: 						&& (empty($_SESSION['id_verification_status']) || $_SESSION['id_verification_status'] == 1)))
2227: 					{
2228: 
2229: 						if (empty($affected_fields)) 
2230: 						{
2231: 
2232: 							return $this->showError('404', 'error', 'allowed data not found');
2233: 						}
2234: 						if (!empty($affected_keys['registration_plate']))
2235: 						{
2236: 							if (empty($filtered_data['license_plate'])) 
2237: 							{
2238: 								return $this->showError('404', 'error', 'empty new registration plate');
2239: 							}
2240: 							if ($d['license_plate'] == $filtered_data['license_plate']) 
2241: 							{
2242: 								unset($filtered_data['license_plate']);
2243: 							}
2244: 						}						
2245: 
2246: 						$car = $this->editCar($filtered_data,$id_car,"",$langs,$order_location, $currency,$country,$region,$city);
2247: 
2248: 						if (!empty($car['error'])) 
2249: 						{
2250: 							return $this->showError('404', 'error', $car['error']);
2251: 						}
2252: 					}
2253: 					else
2254: 					{
2255: 						return $this->showError('404', 'error', 'not enough rights or wrong user role');
2256: 					}
2257: 				}
2258: 				else
2259: 				{
2260: 					$u_id_arr = explode(',',$d['u_id']); sort($u_id_arr);
2261: 					$id_user_arr = explode(',',$id_user); sort($id_user_arr);
2262: 
2263: 					if ($u_id_arr == $id_user_arr)
2264: 					{
2265: 						if (empty($affected_fields)) 
2266: 						{
2267: 							return $this->showError('404', 'error', 'reassign and allowed data not found');
2268: 						}
2269: 						if (!empty($affected_keys['registration_plate'])) 
2270: 						{
2271: 							if (empty($filtered_data['license_plate']))
2272: 							{
2273: 								return $this->showError('404', 'error', 'new empty registration plate');
2274: 							}
2275: 							if ($d['license_plate'] == $filtered_data['license_plate']) 
2276: 							{
2277: 								unset($filtered_data['license_plate']);
2278: 							}
2279: 						}
2280: 
2281: 						$car = $this->editCar($filtered_data,$id_car,"",$langs,$order_location, $currency,$country,$region,$city);
2282: 						if (!empty($car['error'])) 
2283: 						{
2284: 							return $this->showError('404', 'error', $car['error']);
2285: 						}
2286: 					}
```

### `controlTask` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `22895-22924`
```php
22895: 		public function controlTask($id_task_list = '', $id_task_action_control = '')
22896: 		{
22897: 			if (empty($_SESSION[UID])) {
22898: 				return $this->showError('404', 'error', 'unauthorized access');
22899: 			}
22900: 
22901: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22902: 
22903: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
22904: 
22905: 			if (empty($id_task_action_control)) return $this->showError('404', 'error', 'empty action');
22906: 			
22907: 			$s = "INSERT INTO `task_list_action_control`
22908: 				SET 
22909: 				  `id_task_list` = '$id_task_list',
22910: 				  `id_task_action_control` = '$id_task_action_control',
22911: 				  `last_edit_timestamp` = now(),
22912: 				  `create_timestamp` = now(),
22913: 				  `last_edit_datetime` = now(),
22914: 				  `create_datetime` = now()
22915: 				";
22916: 
22917: 			$q = query($s);
22918: 			if ($q === false) return $this->showError('404', 'error', "insert failed");	
22919: 
22920: 			return array(
22921: 				'code' 		=>	'200',
22922: 				'status' 	=>	'success'
22923: 			);
22924: 		}
```

### `createContact` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `21107-21307`
```php
21107: 		public function createContact($data = '', $langs = array())
21108: 		{	
21109: 			if (empty($_SESSION[UID])) {
21110: 				return $this->showError('404', 'error', 'unauthorized access');
21111: 			}
21112: 
21113: 			if (empty($data)) 
21114: 			{
21115: 				return $this->showError('404', 'error', 'empty data');
21116: 			}
21117: 
21118: 			$data = json_decode($data,true);
21119: 			
21120: 			if (empty($data) || !is_array($data)) 
21121: 			{
21122: 				return $this->showError('404', 'error', 'wrong data');
21123: 			}
21124: 
21125: 			$allowed_fields = array(
21126: 				'owner'	=>		array(
21127: 										'name'	=>	'owner',
21128: 										'NULL'	=>	false
21129: 									),
21130: 				'o_type'	=>		array(
21131: 										'name'	=>	'id_owner_type',
21132: 										'NULL'	=>	false
21133: 									),
21134: 				'co_class'	=>		array(
21135: 										'name'	=>	'id_contact_type',
21136: 										'NULL'	=>	false
21137: 									),
21138: 				'number'	=>	array(
21139: 										'name'	=>	'contact_number',
21140: 										'NULL'	=>	true
21141: 									),
21142: 				'cid'	=>		array(
21143: 										'name'	=>	'contact_id',
21144: 										'NULL'	=>	true
21145: 									),
21146: 				'link'	=>		array(
21147: 										'name'	=>	'contact_link',
21148: 										'NULL'	=>	true
21149: 									),
21150: 				'is_bot'	=>		array(
21151: 										'name'	=>	'is_bot',
21152: 										'NULL'	=>	false,
21153: 										'format'=>	function($val){return empty($val) ? 0 : 1;}
21154: 									),
21155: 				'key1'	=>		array(
21156: 										'name'	=>	'key1',
21157: 										'NULL'	=>	true
21158: 									),
21159: 				'key2'	=>		array(
21160: 										'name'	=>	'key2',
21161: 										'NULL'	=>	true
21162: 									),
21163: 				'token'	=>		array(
21164: 										'name'	=>	'token',
21165: 										'NULL'	=>	true
21166: 									),
21167: 				'hash'	=>		array(
21168: 										'name'	=>	'hash',
21169: 										'NULL'	=>	true
21170: 									),
21171: 				'secret'	=>		array(
21172: 										'name'	=>	'secret',
21173: 										'NULL'	=>	true
21174: 									),
21175: 				'host'	=>		array(
21176: 										'name'	=>	'host',
21177: 										'NULL'	=>	true
21178: 									),	
21179: 				'port'	=>		array(
21180: 										'name'	=>	'port',
21181: 										'NULL'	=>	true
21182: 									),
21183: 				'login'	=>		array(
21184: 										'name'	=>	'login',
21185: 										'NULL'	=>	true
21186: 									),
21187: 				'password'	=>		array(
21188: 										'name'	=>	'password',
21189: 										'NULL'	=>	true
21190: 									),
21191: 				'smtpsecure'	=>		array(
21192: 										'name'	=>	'smtpsecure',
21193: 										'NULL'	=>	true
21194: 									),
21195: 				'fromname'	=>		array(
21196: 										'name'	=>	'fromname',
21197: 										'NULL'	=>	true
21198: 									),			
21199: 				'active'	=>		array(
21200: 										'name'	=>	'active',
21201: 										'NULL'	=>	false,
21202: 										'format'=>	function($val){return empty($val) ? 0 : 1;}
21203: 									)
21204: 			);
21205: 			foreach ($langs as $lang)
21206: 			{
21207: 				$out_key = $lang['iso'];
21208: 				$db_key = 'name_' . $lang['iso'];
21209: 				$null_on = true;
21210: 				$allowed_fields[$out_key] = array(
21211: 									'name'	=>	$db_key ,
21212: 									'NULL'	=>	$null_on
21213: 				);				
21214: 				$out_key = 'about_' . $lang['iso'];
21215: 				$db_key = 'description_' . $lang['iso'];
21216: 				$null_on = false;
21217: 				$allowed_fields[$out_key] = array(
21218: 									'name'	=>	$db_key ,
21219: 									'NULL'	=>	$null_on
21220: 				);
21221: 			}
21222: 
21223: 			if (!isset($data['co_class'])) return $this->showError('404', 'error', 'empty co_class');
21224: 			$forbidden_fields = array();
21225: 			$affected_fields = array();
21226: 			$filtered_data = array();
21227: 			foreach ($data as $key => $value)
21228: 			{
21229: 				if (isset($allowed_fields[$key]))
21230: 				{
21231: 					$affected_fields[] = $key;
21232: 					if (!empty($allowed_fields[$key]['format']))
21233: 					{
21234: 						$value = $allowed_fields[$key]['format']($value);
21235: 					}
21236: 					if (empty($value['error']))
21237: 					{
21238: 						$name = $allowed_fields[$key]['name'];
21239: 						$null_on = $allowed_fields[$key]['NULL'];
21240: 						if ($null_on === true || $value !== NULL)
21241: 						{
21242: 							$filtered_data[$name] = $value;
21243: 						}
21244: 						else
21245: 						{
21246: 							return $this->showError('404', 'error', "$key: null value");
21247: 						}
21248: 					}
21249: 					else
21250: 					{
21251: 						return $this->showError('404', 'error', "$key: {$value['error']}");
21252: 					}
21253: 				}
21254: 				else
21255: 				{
21256: 					$forbidden_fields[] = $key;
21257: 				}
21258: 			}
21259: 
21260: 			if (isset($filtered_data['owner'])) 
21261: 			{
21262: 				if (isset($filtered_data['id_owner_type']) && $filtered_data['id_owner_type'] != 1) 
21263: 				{
21264: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
21265: 				}
21266: 				else
21267: 				{
21268: 					if ($filtered_data['owner'] != $_SESSION[UID] && $this->id_role != 4) return $this->showError('404', 'error', 'not enough rights for action');
21269: 				}
21270: 			}
21271: 			else
21272: 			{
21273: 				if (isset($filtered_data['id_owner_type']) && $filtered_data['id_owner_type'] != 1) return $this->showError('404', 'error', 'empty owner');
21274: 				$filtered_data['owner'] = $_SESSION[UID];
21275: 				$affected_fields[] = 'owner';
21276: 			}
21277: 
21278: 			$s = array();
21279: 			foreach ($filtered_data as $key => $value)
21280: 			{
21281: 				$s[] = "`$key` = " 
21282: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
21283: 			}
21284: 
21285: 			$s = "INSERT INTO `contact_items`
21286: 				SET 
21287: 					" . implode(",\n", $s) .",
21288: 					`last_edit_datetime` = now(),
21289: 					`last_edit_user` = '" . $_SESSION[UID] . "',
21290: 					`create_datetime` = now(),
21291: 					`create_user` = '" . $_SESSION[UID] . "'
21292: 				";
21293: 
21294: 			$q = query($s);
21295: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');
21296: 
21297: 			$id_contact_item = insert_id();
21298: 			$out = array('affected_fields' 	=> 	$affected_fields);
21299: 			if (!empty($forbidden_fields)) $out['forbidden_fields'] = $forbidden_fields;
21300: 			$out['co_id'] = $id_contact_item;
21301: 
21302: 			return array(
21303: 				'code' 		=>	'200',
21304: 				'status' 	=>	'success',
21305: 				'data' 		=>	$out
21306: 			);
21307: 		}
```

### `createDropboxFile` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `13712-13989`
```php
13712: 		public function createDropboxFile($file = '')
13713: 		{
13714: 			if (empty($_SESSION[UID])) {
13715: 				return $this->showError('404', 'error', 'unauthorized access');
13716: 			}
13717: 
13718: 			@$file = json_decode($file,true);
13719: 		
13720: 			if (empty($file) || !is_array($file)) 
13721: 			{
13722: 				return $this->showError('404', 'error', 'wrong file data');
13723: 			}
13724: 			$id_user = $_SESSION[UID];
13725: 			if (isset($file['u_id']))
13726: 			{
13727: 				$id_user = trim($file['u_id']);	
13728: 				if ($id_user != $_SESSION[UID])
13729: 				{
13730: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
13731: 
13732: 					$s = "SELECT
13733: 							`id_user`
13734: 						FROM `users` 		
13735: 						WHERE	
13736: 							`id_user` = '" . $id_user. "'
13737: 						LIMIT 1
13738: 						";
13739: 
13740: 					$q = query($s);
13741: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
13742: 
13743: 					$d = fetch_assoc($q);
13744: 					if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');	
13745: 				}
13746: 			}
13747: 
13748: 			if (isset($file['dl_id']))
13749: 			{
13750: 				$id_dropbox_link = $file['dl_id'];
13751: 				if ($this->id_role == 4)
13752: 				{
13753: 					$sql = '';
13754: 					if (isset($file['u_id']))
13755: 					{
13756: 						$sql = ",
13757: 							IF(`deleted` = 0,(
13758: 								SELECT
13759: 									`id_user`
13760: 								FROM
13761: 									`users_dropbox_link`
13762: 								WHERE
13763: 									`users_dropbox_link`.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link` AND 
13764: 									`id_user` = '$id_user'
13765: 							),'') as user";
13766: 					}
13767: 					$s = "SELECT
13768: 							`id_dropbox_link`,
13769: 							`json`,
13770: 							`private`,
13771: 							`deleted`$sql
13772: 						FROM `dropbox_link` 				
13773: 						WHERE	
13774: 							`id_dropbox_link` = '$id_dropbox_link'
13775: 						LIMIT 1
13776: 						";
13777: 
13778: 					$q = query($s);
13779: 					if ($q === false) return $this->showError('404', 'error', 'database select failed');
13780: 
13781: 					$d = fetch_assoc($q);
13782: 					if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link not found');
13783: 					if (!empty($d['deleted']))return $this->showError('404', 'error', 'deleted dropbox link');
13784: 					if (isset($file['u_id']) && !empty($d['user'])) unset($file['u_id']);
13785: 				}
13786: 				else
13787: 				{
13788: 					$s = "SELECT
13789: 							`id_dropbox_link`,
13790: 							`json`,
13791: 							`private`,
13792: 							`deleted`,
13793: 							IF(`deleted` = 0,(
13794: 								SELECT
13795: 									`id_user`
13796: 								FROM
13797: 									`users_dropbox_link`
13798: 								WHERE
13799: 									`users_dropbox_link`.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link` AND 
13800: 									`id_user` = '{$_SESSION[UID]}'
13801: 							),'') as user
13802: 						FROM `dropbox_link` 				
13803: 						WHERE	
13804: 							`id_dropbox_link` = '$id_dropbox_link'
13805: 						LIMIT 1
13806: 						";
13807: 
13808: 					$q = query($s);
13809: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
13810: 
13811: 					$d = fetch_assoc($q);
13812: 					if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link for dl_id not found');
13813: 					if (!empty($d['deleted']))return $this->showError('404', 'error', 'dropbox link is deleted');
13814: 					if (empty($d['user']))
13815: 					{
13816: 						return $this->showError('404', 'error', 'not enough rights');
13817: 					}
13818: 					unset($file['u_id']);
13819: 				}
13820: 				$json = json_decode($d['json'],true);
13821: 				$name_upload = $json['name_upload'];
13822: 				
13823: 				if (isset($file['private']))
13824: 				{
13825: 					$file['private'] = (int)$file['private'];
13826: 					if ($file['private'] == $d['private']) unset($file['private']);
13827: 				}
13828: 			}
13829: 
13830: 			if (isset($file['base64']))
13831: 			{
13832: 				@$content = (string)$file['base64'];
13833: 				list($type, $content) = array_merge(explode(';', $content),array(''));
13834: 				list(, $type) = array_merge(explode(':', $type),array(''));
13835: 				if (empty($type)) return $this->showError('404', 'error', 'wrong file string');
13836: 				list($base64, $content) = array_merge(explode(',', $content),array(''));
13837: 				if ($base64 !== 'base64') return $this->showError('404', 'error', 'wrong string of file');
13838: 				$content = base64_decode($content);
13839: 				if (gettype($content) !== 'string') return $this->showError('404', 'error', 'wrong base64');
13840: 			}
13841: 			else
13842: 			{
13843: 				if (isset($file['dl_id']))
13844: 				{
13845: 					if (!isset($file['private']) && !isset($file['u_id'])) return $this->showError('404', 'error', 'new data not found');
13846: 				}
13847: 				else
13848: 				{
13849: 					return $this->showError('404', 'error', 'base64 key not found');
13850: 				}
13851: 			}
13852: 
13853: 			if (isset($file['dl_id']))
13854: 			{
13855: 				$s = array();
13856: 				if (isset($content)) 
13857: 				{
13858: 					$json['type'] = $type;
13859: 					$json['size'] = strlen($content);
13860: 					$response = upload_to_dropbox($content,$name_upload,$id_dropbox_link,'overwrite');			
13861: 					if (!empty($response['error'])) return $this->showError('404', 'error', array('response' => $response['error']));				
13862: 					$json['response'] = $response['data'];
13863: 					$s[] = "`json` = '" . real_escape_string(json_encode($json)) . "'";
13864: 				}
13865: 				if (isset($file['private'])) $s[] = "`private` = '{$file['private']}'";
13866: 			
13867: 				if (!empty($s))
13868: 				{
13869: 					$s = implode(',',$s);
13870: 					$s = "UPDATE `dropbox_link`
13871: 						SET
13872: 							$s
13873: 						WHERE
13874: 							`id_dropbox_link` = '" . $id_dropbox_link . "'
13875: 						";
13876: 
13877: 					$q = query($s);
13878: 
13879: 					$qs = affected_rows();
13880: 					if ($qs === -1) 
13881: 					{
13882: 						$warning = 'update query failed';
13883: 					}
13884: 					elseif ($qs === 0) 
13885: 					{
13886: 						$warning = 'modified data not found for query';
13887: 					}
13888: 				}
13889: 
13890: 				if (isset($file['u_id']))
13891: 				{				
13892: 					$s = "DELETE
13893: 						FROM `users_dropbox_link`
13894: 						WHERE 
13895: 							`id_dropbox_link` = ' $id_dropbox_link'
13896: 						";
13897: 					$q = query($s);
13898: 
13899: 					if ($q === false) return $this->showError('404', 'error', 'delete failed');
13900: 
13901: 					$s = "INSERT INTO `users_dropbox_link`
13902: 						SET 
13903: 							`id_user` = '$id_user',
13904: 							`id_dropbox_link` = ' $id_dropbox_link'
13905: 						";
13906: 
13907: 					$q = query($s);
13908: 
13909: 					if ($q === false) return $this->showError('404', 'error', 'binding failed');
13910: 				}
13911: 			}
13912: 			else
13913: 			{
13914: 				$filename_upload = 'file';
13915: 				$filename = '';
13916: 				if (isset($file['name']))
13917: 				{
13918: 					@$filename = (string)$file['name'];
13919: 					if (strlen($filename) !== 0) 
13920: 					{
13921: 						$filename_upload = substr($filename ,0,115);
13922: 					}
13923: 				}
13924: 				$sql = "";
13925: 				if (isset($file['private']))
13926: 				{
13927: 					$sql = ",`private` = '{$file['private']}'";
13928: 				}
13929: 				$json = array('name' => $filename, 'name_upload' => $filename_upload, 'type' => $type, 'size' => strlen($content));
13930: 
13931: 				$q = query("BEGIN");
13932: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
```

### `createOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 4=Administrator, 5=Agent`
- source range: `2844-3344`
```php
2844: 		public function createOrder($data = array(), $langs = array(), $payment_method = array(), $payment_card = array(), $comment_items = array(), $order_service = array(), $contact = array(), $order_location = array(), $currency = array(), $unit_set = array(), $country = array(),$region = array(),$city = array(), $schedule = array(), $price_time_functions = array(), $stripe_seat_title_template = '', $stripe_request_duration = 0, $aggregators = array())
2845: 		{
2846: 			if (empty($_SESSION[UID])) {
2847: 				return $this->showError('404', 'error', 'unauthorized access');
2848: 			}
2849: 			if ($this->id_role != 1 && $this->id_role != 5 && $this->id_role != 4)
2850: 			{
2851: 				return $this->showError('404', 'error', 'wrong user role');
2852: 			}
2853: 			if (!empty($_SESSION['user_ban_status']['order']))
2854: 			{
2855: 				return $this->showError('404', 'error', 'user banned');
2856: 			}	
2857: 			
2858: 			if (empty($data)) 
2859: 			{
2860: 				return $this->showError('404', 'error', 'empty data');
2861: 			}
2862: 
2863: 			$data = json_decode($data,true);
2864: 			
2865: 			if (empty($data) || !is_array($data)) 
2866: 			{
2867: 				return $this->showError('404', 'error', 'wrong data');
2868: 			}
2869: 			$iso = array();
2870: 			foreach ($langs as $lang)
2871: 			{
2872: 				$iso[$lang['iso']] = array(
2873: 										'name'				=>$lang['iso'],
2874: 										'type'				=>'string',
2875: 										'NULL'				=>false,
2876: 										'values'			=>NULL,
2877: 										'default'			=>NULL
2878: 				);
2879: 			}
2880: 			$options_valid_keys = array();
2881: 			foreach ($this->constant['b_options_valid_keys'] as $key => $value)
2882: 			{
2883: 				$options_valid_keys[$key] = array(
2884: 										'name'				=>$key,
2885: 										'type'				=>'string',
2886: 										'NULL'				=>false,
2887: 										'values'			=>NULL,
2888: 										'default'			=>NULL
2889: 				);
2890: 			}
2891: 			foreach ($contact as $key => $value)
2892: 			{
2893: 				$contact[$key] = array(
2894: 										'name'				=>$key,
2895: 										'type'				=>'string',
2896: 										'NULL'				=>false,
2897: 										'values'			=>NULL,
2898: 										'default'			=>NULL
2899: 				);
2900: 			}
2901: 		
2902: 			$allowed_fields = array(						
2903: 				'b_start_address'			=>		array(
2904: 							'name'				=>'from',
2905: 							'type'				=>'string',
2906: 							'NULL'				=>false,
2907: 							'values'			=>NULL,
2908: 							'default'			=>array(""),
2909: 							'error'				=>array(
2910: 								'requirement'     =>array(
2911: 									array('empty','from'),
2912: 									array('&&'),
2913: 									array(
2914: 										array('!isset','from_lat'),
2915: 										array('||'),
2916: 										array('!isset','from_lng'),
2917: 									)
2918: 								),
2919: 								'msg' 			=>'empty start address'
2920: 							)
2921: 				),
2922: 				'b_start_latitude'			=>		array(
2923: 							'name'				=>'from_lat',
2924: 							'type'				=>'latitude',
2925: 							'NULL'				=>true,
2926: 							'values'			=>NULL,
2927: 							'default'			=>array(NULL)
2928: 				),
2929: 				'b_start_longitude'			=>		array(
2930: 							'name'				=>'from_lng',
2931: 							'type'				=>'longitude',
2932: 							'NULL'				=>true,
2933: 							'values'			=>NULL,
2934: 							'default'			=>array(NULL)
2935: 				),
2936: 				'b_destination_address'		=>		array(
2937: 							'name'				=>'to',
2938: 							'type'				=>'string',
2939: 							'NULL'				=>false,
2940: 							'values'			=>NULL,
2941: 							'default'			=>array("")
2942: 				),
2943: 				'b_destination_latitude'	=>		array(
2944: 							'name'				=>'to_lat',
2945: 							'type'				=>'latitude',
2946: 							'NULL'				=>true,
2947: 							'values'			=>NULL,
2948: 							'default'			=>array(NULL)
2949: 				),
2950: 				'b_destination_longitude'	=>		array(
2951: 							'name'				=>'to_lng',
2952: 							'type'				=>'longitude',
2953: 							'NULL'				=>true,
2954: 							'values'			=>NULL,
2955: 							'default'			=>array(NULL)
2956: 				),
2957: 				'b_start_datetime'			=>		array(
2958: 							'name'				=>'datetime_start_plan',
2959: 							'type'				=>'datetime',
2960: 							'NULL'				=>false,
2961: 							'values'			=>NULL,
2962: 							'default'			=>array('0000-00-00 00:00:00'),
2963: 							'error'				=>array(
2964: 								'requirement'     =>array(
2965: 									array('empty','datetime_start_plan')
2966: 								),
2967: 								'msg' 			=>'empty booking datetime'
2968: 							),
2969: 							'var'				=>array(
2970: 								'before' 			=>array(
2971: 									array('max_waiting_time',$this->constant['waiting_interval'])
2972: 								),
2973: 								'any'				=>array(
2974: 									array('max_waiting_time',$this->constant['waiting_interval_any'])
2975: 								)
2976: 							)
2977: 				),
2978: 				'b_custom_comment'			=>		array(
2979: 							'name'				=>'comment',
2980: 							'type'				=>'string',
2981: 							'NULL'				=>false,
2982: 							'values'			=>NULL,
2983: 							'default'			=>array("")
2984: 				),
2985: 				'b_flight_number'			=>		array(
2986: 							'name'				=>'flight_number',
2987: 							'type'				=>'string',
2988: 							'NULL'				=>false,
2989: 							'values'			=>NULL,
2990: 							'default'			=>array("")
2991: 				),
2992: 				'b_terminal'				=>		array(
2993: 							'name'				=>'terminal',
2994: 							'type'				=>'string',
2995: 							'NULL'				=>false,
2996: 							'values'			=>NULL,
2997: 							'default'			=>array("")
2998: 				),
2999: 				'b_passengers_count'		=>		array(
3000: 							'name'				=>'passenger_count',
3001: 							'type'				=>'unsigned integer',
3002: 							'NULL'				=>false,
3003: 							'values'			=>NULL,
3004: 							'default'			=>array(0)
3005: 				),
3006: 				'b_luggage_count'			=>		array(
3007: 							'name'				=>'luggage_count',
3008: 							'type'				=>'unsigned integer',
3009: 							'NULL'				=>false,
3010: 							'values'			=>NULL,
3011: 							'default'			=>array(0)
3012: 				),
3013: 				'b_placard'					=>		array(
3014: 							'name'				=>'placard',
3015: 							'type'				=>'string',
3016: 							'NULL'				=>false,
3017: 							'values'			=>NULL,
3018: 							'default'			=>array("")
3019: 				),
3020: 				'b_car_class'				=>		array(
3021: 							'name'				=>'id_car_class',
3022: 							'type'				=>'integer',
3023: 							'NULL'				=>true,
3024: 							'values'			=>$this->car_class,
3025: 							'default'			=>array(NULL),
3026: 							'error'				=>array(
3027: 								'requirement'     =>array(
3028: 									array('isset','id_car_class'),
3029: 									array('&&'),
3030: 									array('empty_values','id_car_class')
3031: 								),
3032: 								'msg' 			=>'wrong car class'
3033: 							)			
3034: 				),
3035: 				'b_payment_way'				=>		array(
3036: 							'name'				=>'id_payment_method',
3037: 							'type'				=>'integer',
3038: 							'NULL'				=>false,
3039: 							'values'			=>$payment_method,
3040: 							'default'			=>array(0),
3041: 							'error'				=>array(
3042: 								'requirement'     =>array(
3043: 									array('empty','id_payment_method')
3044: 								),
3045: 								'msg' 			=>'empty payment way'
3046: 							)			
3047: 				),
3048: 				'b_payment_card'			=>		array(
3049: 							'name'				=>'id_payment_card',
3050: 							'type'				=>'integer',
3051: 							'NULL'				=>true,
3052: 							'values'			=>$payment_card,
3053: 							'default'			=>array(NULL) 
3054: 				),
3055: 				'b_cars_count'				=>		array(
3056: 							'name'				=>'car_count',
3057: 							'type'				=>'unsigned integer',		
3058: 							'NULL'				=>false,
3059: 							'values'			=>NULL,
3060: 							'default'			=>array(1)
3061: 				),
3062: 				'b_max_waiting'				=>		array(
3063: 							'name'				=>'max_waiting_datetime',
3064: 							'type'				=>'second to datetime',
```

### `createPayment` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `24668-24862`
```php
24668: 		public function createPayment($data = '', $appUrl = '', $payment_services = array(), $payment_ways = array())
24669: 		{
24670: 			if (empty($_SESSION[UID])) {
24671: 				return $this->showError('404', 'error', 'unauthorized access');
24672: 			}
24673: 
24674: 			if (empty($data)) 
24675: 			{
24676: 				return $this->showError('404', 'error', 'empty data');
24677: 			}
24678: 
24679: 			$data = json_decode($data,true);
24680: 			
24681: 			if (empty($data) || !is_array($data)) 
24682: 			{
24683: 				return $this->showError('404', 'error', 'wrong data');
24684: 			}
24685: 
24686: 			$allowed_fields = array(
24687: 				'sum'	=>		array(
24688: 										'name'	=>	'sum',
24689: 										'NULL'	=>	true
24690: 									),
24691: 				'currency'	=>		array(
24692: 										'name'	=>	'currency',
24693: 										'NULL'	=>	true
24694: 									),
24695: 				'payment_service'	=>		array(
24696: 										'name'	=>	'id_payment_service',
24697: 										'NULL'	=>	true
24698: 									),
24699: 				'subs_id'	=>		array(
24700: 										'name'	=>	'id_user_subscription',
24701: 										'NULL'	=>	true
24702: 									),
24703: 				'payment_way'	=>		array(
24704: 										'name'	=>	'id_payment_method',
24705: 										'NULL'	=>	true
24706: 									),
24707: 				'from'	=>		array(
24708: 										'name'	=>	'from',
24709: 										'NULL'	=>	true
24710: 									),
24711: 				'json'	=>		array(
24712: 										'name'	=>	'json',
24713: 										'NULL'	=>	false
24714: 									)
24715: 			);
24716: 
24717: 			$forbidden_fields = array();
24718: 			$affected_fields = array();
24719: 			$filtered_data = array();
24720: 			$add_table_list = array();
24721: 			foreach ($data as $key => $value)
24722: 			{
24723: 				if (isset($allowed_fields[$key]))
24724: 				{
24725: 					if (!empty($allowed_fields[$key]['format']))
24726: 					{
24727: 						$value = $allowed_fields[$key]['format']($value);
24728: 					}
24729: 					if (empty($value['error']))
24730: 					{
24731: 						if ($allowed_fields[$key]['name'] === NULL)
24732: 						{
24733: 							$add_table_list[] = array(
24734: 								$allowed_fields[$key]['table'],
24735: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
24736: 								$value,
24737: 								$key,
24738: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
24739: 								empty($data['exact']) ? false : true
24740: 							);
24741: 						}
24742: 						else
24743: 						{									
24744: 							$name = $allowed_fields[$key]['name'];
24745: 							$null_on = $allowed_fields[$key]['NULL'];
24746: 							if ($null_on === true || $value !== NULL)
24747: 							{
24748: 								$affected_fields[] = $key;
24749: 								$filtered_data[$name] = $value;
24750: 							}
24751: 							else
24752: 							{
24753: 								return $this->showError('404', 'error', "$key: null value");
24754: 							}
24755: 						}
24756: 					}
24757: 					else
24758: 					{
24759: 						return $this->showError('404', 'error', "$key: {$value['error']}");
24760: 					}
24761: 				}
24762: 				else
24763: 				{
24764: 					$forbidden_fields[] = $key;
24765: 				}
24766: 			}
24767: 
24768: 			if (empty($filtered_data['sum'])) return $this->showError('404', 'error', 'empty sum');
24769: 			if (empty($filtered_data['currency'])) $filtered_data['currency'] = DEFAULT_CURRENCY;		
24770: 			if (array_key_exists('from',$filtered_data))
24771: 			{
24772: 				if ($this->id_role != 4) 
24773: 				{
24774: 					return $this->showError('404', 'error', 'not enough rights');
24775: 				}
24776: 			}
24777: 			else
24778: 			{
24779: 				$filtered_data['from'] = $_SESSION[UID];
24780: 			}
24781: 
24782: 			if (empty($filtered_data['id_payment_service'])) return $this->showError('404', 'error', 'empty payment service');
24783: 			if (empty($payment_services[$filtered_data['id_payment_service']])) return $this->showError('404', 'error', 'wrong payment service');
24784: 			if (empty($payment_services[$filtered_data['id_payment_service']]['var'])) return $this->showError('404', 'error', 'empty payment service var');
24785: 			$f_name = "{$payment_services[$filtered_data['id_payment_service']]['var']}_create_payment";		
24786: 			if (!function_exists($f_name)) return $this->showError('404', 'error', 'function not found');
24787: 			if (empty($filtered_data['id_payment_method'])) $filtered_data['id_payment_method'] = 2;
24788: 			if (empty($payment_ways[$filtered_data['id_payment_method']])) return $this->showError('404', 'error', 'wrong payment way');			
24789: 			foreach($payment_services[$filtered_data['id_payment_service']]['payment_ways'] as $key=>$val)
24790: 			{
24791: 				if ($val[0] == $filtered_data['id_payment_method']) 
24792: 				{
24793: 					$p_w_index = $key;	
24794: 					if (!empty($val[1]['api']['type'])) $p_w_type =  $val[1]['api']['type'];
24795: 				}
24796: 			}
24797: 			if (!isset($p_w_index)) return $this->showError('404', 'error', 'forbidden payment way');
24798: 			if (!isset($p_w_type)) return $this->showError('404', 'error', 'empty payment way type');
24799: 	
24800: 
24801: 			$s = array();
24802: 			foreach ($filtered_data as $key => $value)
24803: 			{
24804: 				$s[] = "`$key` = " 
24805: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
24806: 			}
24807: 
24808: 			$q = query("BEGIN");
24809: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
24810: 
24811: 			$now_time = time();
24812: 			$s = "INSERT INTO `payment`
24813: 				SET 
24814: 					" . implode(",\n", $s) .",
24815: 				  `create_user` = '{$_SESSION[UID]}',
24816: 				  `last_edit_user` = '{$_SESSION[UID]}',
24817: 				  `last_edit_int_timestamp` = '$now_time',
24818: 				  `create_int_timestamp` = '$now_time'
24819: 				";
24820: 
24821: 			$q = query($s);
24822: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
24823: 			
24824: 			$p_id = insert_id();
24825: 			$create_status = $f_name($filtered_data['sum'],$filtered_data['currency'],$appUrl,'',$p_id,$p_w_type);
24826: 			if (!empty($create_status['error'])) return $this->showError('404', 'error', $create_status['error']);
24827: 			
24828: 			$s = "UPDATE `payment`
24829: 				SET
24830: 					`json` = '" . real_escape_string(json_encode(array('confirmation_url'=>$create_status['confirmation_url']))) . "',
24831: 					`last_edit_int_timestamp` = '" . time() . "',
24832: 					`id_outer` = '{$create_status['id']}'
24833: 				WHERE
24834: 					`id_payment` = '$p_id'
24835: 				";
24836: 
24837: 			$q = query($s);
24838: 				
24839: 			$update = affected_rows();
24840: 			if ($update === -1) 
24841: 			{
24842: 				return $this->showError('400','error','update failed'. error_db());
24843: 			}
24844: 			elseif ($update === 0) 
24845: 			{
24846: 				return $this->showError('400','error','modified data not found');
24847: 			}
24848: 			
24849: 			$q = query("COMMIT");
24850: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
24851: 
24852: 			$out = array('affected_fields' 	=> 	$affected_fields);
24853: 			if (!empty($forbidden_fields)) $out['forbidden_fields'] = $forbidden_fields;
24854: 			$out['p_id'] = $p_id;
24855: 			$out['confirmation_url'] = $create_status['confirmation_url'];
24856: 
24857: 			return array(
24858: 				'code' 		=>	'200',
24859: 				'status' 	=>	'success',		
24860: 				'data' 		=>	$out
24861: 			);
24862: 		}
```

### `createSubscription` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `24967-25101`
```php
24967: 		public function createSubscription($data = '')
24968: 		{
24969: 			if (empty($_SESSION[UID])) {
24970: 				return $this->showError('404', 'error', 'unauthorized access');
24971: 			}
24972: 
24973: 			if (empty($data)) 
24974: 			{
24975: 				return $this->showError('404', 'error', 'empty data');
24976: 			}
24977: 
24978: 			$data = json_decode($data,true);
24979: 			
24980: 			if (empty($data) || !is_array($data)) 
24981: 			{
24982: 				return $this->showError('404', 'error', 'wrong data');
24983: 			}
24984: 
24985: 			$allowed_fields = array(
24986: 				'u_id'	=>		array(
24987: 										'name'	=>	'id_user',
24988: 										'NULL'	=>	false
24989: 									),
24990: 				'tariff'	=>		array(
24991: 										'name'	=>	'id_tariff',
24992: 										'NULL'	=>	false
24993: 									),
24994: 				'start_date'	=>		array(
24995: 										'name'	=>	'start_date',
24996: 										'NULL'	=>	false
24997: 									),
24998: 				'end_date'	=>		array(
24999: 										'name'	=>	'end_date',
25000: 										'NULL'	=>	false
25001: 									),
25002: 				'subs_status'	=>		array(
25003: 										'name'	=>	'id_subscription_status',
25004: 										'NULL'	=>	true
25005: 									),
25006: 				'auto_renew'	=>		array(
25007: 										'name'	=>	'auto_renew',
25008: 										'NULL'	=>	false
25009: 									)
25010: 			);
25011: 
25012: 			$forbidden_fields = array();
25013: 			$affected_fields = array();
25014: 			$filtered_data = array();
25015: 			$add_table_list = array();
25016: 			foreach ($data as $key => $value)
25017: 			{
25018: 				if (isset($allowed_fields[$key]))
25019: 				{
25020: 					if (!empty($allowed_fields[$key]['format']))
25021: 					{
25022: 						$value = $allowed_fields[$key]['format']($value);
25023: 					}
25024: 					if (empty($value['error']))
25025: 					{
25026: 						if ($allowed_fields[$key]['name'] === NULL)
25027: 						{
25028: 							$add_table_list[] = array(
25029: 								$allowed_fields[$key]['table'],
25030: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
25031: 								$value,
25032: 								$key,
25033: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
25034: 								empty($data['exact']) ? false : true
25035: 							);
25036: 						}
25037: 						else
25038: 						{									
25039: 							$name = $allowed_fields[$key]['name'];
25040: 							$null_on = $allowed_fields[$key]['NULL'];
25041: 							if ($null_on === true || $value !== NULL)
25042: 							{
25043: 								$affected_fields[] = $key;
25044: 								$filtered_data[$name] = $value;
25045: 							}
25046: 							else
25047: 							{
25048: 								return $this->showError('404', 'error', "$key: null value");
25049: 							}
25050: 						}
25051: 					}
25052: 					else
25053: 					{
25054: 						return $this->showError('404', 'error', "$key: {$value['error']}");
25055: 					}
25056: 				}
25057: 				else
25058: 				{
25059: 					$forbidden_fields[] = $key;
25060: 				}
25061: 			}
25062: 
25063: 			if (empty($filtered_data['id_tariff'])) return $this->showError('404', 'error', 'empty tariff');
25064: 			if ($this->id_role != 4) 
25065: 			{
25066: 				while (true)
25067: 				{
25068: 					if (!array_key_exists('id_user',$filtered_data)) break;
25069: 					if (!array_key_exists('subs_status',$filtered_data)) break;
25070: 					return $this->showError('404', 'error', 'not enough rights');
25071: 				}
25072: 			}
25073: 			if (!array_key_exists('id_user',$filtered_data)) $filtered_data['id_user'] = $_SESSION[UID];
25074: 
25075: 			$s = array();
25076: 			foreach ($filtered_data as $key => $value)
25077: 			{
25078: 				$s[] = "`$key` = " 
25079: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
25080: 			}
25081: 
25082: 			$s = "INSERT INTO `user_subscription`
25083: 				SET 
25084: 					" . implode(",\n", $s) ."
25085: 				";
25086: 
25087: 			$q = query($s);
25088: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
25089: 			
25090: 			$subs_id = insert_id();
25091: 
25092: 			$out = array('affected_fields' 	=> 	$affected_fields);
25093: 			if (!empty($forbidden_fields)) $out['forbidden_fields'] = $forbidden_fields;
25094: 			$out['subs_id'] = $subs_id;
25095: 
25096: 			return array(
25097: 				'code' 		=>	'200',
25098: 				'status' 	=>	'success',		
25099: 				'data' 		=>	$out
25100: 			);
25101: 		}
```

### `createTask` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `22610-22893`
```php
22610: 		public function createTask($data = '')
22611: 		{	
22612: 			if (empty($_SESSION[UID])) {
22613: 				return $this->showError('404', 'error', 'unauthorized access');
22614: 			}
22615: 
22616: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22617: 
22618: 			if (empty($data)) 
22619: 			{
22620: 				return $this->showError('404', 'error', 'empty data');
22621: 			}
22622: 
22623: 			$data = json_decode($data,true);
22624: 			
22625: 			if (empty($data) || !is_array($data)) 
22626: 			{
22627: 				return $this->showError('404', 'error', 'wrong data');
22628: 			}
22629: 
22630: 			$allowed_fields = array(
22631: 				'min_action_interval'	=>		array(
22632: 										'name'	=>	'min_action_interval',
22633: 										'NULL'	=>	false
22634: 									),
22635: 				'max_action_interval'	=>		array(
22636: 										'name'	=>	'max_action_interval',
22637: 										'NULL'	=>	false
22638: 									),
22639: 				'comment'	=>		array(
22640: 										'name'	=>	'comment',
22641: 										'NULL'	=>	false
22642: 									),
22643: 				'account'	=>	array(
22644: 										'name'	=>	'account',
22645: 										'NULL'	=>	false
22646: 									),
22647: 				'json'	=>		array(
22648: 										'name'	=>	'json',
22649: 										'NULL'	=>	false
22650: 									),
22651: 				'active'	=>		array(
22652: 										'name'	=>	'active',
22653: 										'NULL'	=>	false,
22654: 										'format'=>	function($val){return empty($val) ? 0 : 1;}
22655: 									),							
22656: 				'bind_account'	=>		array(
22657: 												'name'	=>	NULL,
22658: 												'NULL'	=>	true,
22659: 												'table'	=>	'task_list_account',
22660: 												'key'	=>	'id_account'
22661: 											),
22662: 				'bind_comment'	=>		array(
22663: 												'name'	=>	NULL,
22664: 												'NULL'	=>	true,
22665: 												'table'	=>	'task_list_comment',
22666: 												'key'	=>	'id_task_comment'
22667: 											),
22668: 				'bind_action_function'	=>		array(
22669: 												'name'	=>	NULL,
22670: 												'NULL'	=>	true,
22671: 												'table'	=>	'task_list_action_function',
22672: 												'allowed_fields'=>	array(
22673: 													'task_action_function'	=>	array(
22674: 																		'name'	=>	'id_task_action_function',
22675: 																		'NULL'	=>	false
22676: 																	),
22677: 													'parameter'	=>	array(
22678: 																		'name'	=>	'parameter',
22679: 																		'NULL'	=>	false,
22680: 																		/*'format'=>	function($val,$id_val){
22681: 																			return array(
22682: 																				'formated'	=>	str_replace(array(chr(0),chr(1),chr(2)),' ',$val)
22683: 																			);
22684: 																		}*/
22685: 																	)
22686: 												)
22687: 											)
22688: 			);
22689: 
22690: 			$forbidden_fields = array();
22691: 			$affected_fields = array();
22692: 			$filtered_data = array();
22693: 			$add_table_list = array();
22694: 			foreach ($data as $key => $value)
22695: 			{
22696: 				if (isset($allowed_fields[$key]))
22697: 				{
22698: 					if (!empty($allowed_fields[$key]['format']))
22699: 					{
22700: 						$value = $allowed_fields[$key]['format']($value);
22701: 					}
22702: 					if (empty($value['error']))
22703: 					{
22704: 						if ($allowed_fields[$key]['name'] === NULL)
22705: 						{
22706: 							$add_table_list[] = array(
22707: 								$allowed_fields[$key]['table'],
22708: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
22709: 								$value,
22710: 								$key,
22711: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
22712: 								empty($data['exact']) ? false : true
22713: 							);
22714: 						}
22715: 						else
22716: 						{									
22717: 							$name = $allowed_fields[$key]['name'];
22718: 							$null_on = $allowed_fields[$key]['NULL'];
22719: 							if ($null_on === true || $value !== NULL)
22720: 							{
22721: 								$affected_fields[] = $key;
22722: 								$filtered_data[$name] = $value;
22723: 							}
22724: 							else
22725: 							{
22726: 								return $this->showError('404', 'error', "$key: null value");
22727: 							}
22728: 						}
22729: 					}
22730: 					else
22731: 					{
22732: 						return $this->showError('404', 'error', "$key: {$value['error']}");
22733: 					}
22734: 				}
22735: 				else
22736: 				{
22737: 					$forbidden_fields[] = $key;
22738: 				}
22739: 			}
22740: 
22741: 			$s = array();
22742: 			$filtered_data['id_task_status'] = 1;
22743: 			foreach ($filtered_data as $key => $value)
22744: 			{
22745: 				$s[] = "`$key` = " 
22746: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
22747: 			}
22748: 
22749: 			$q = query("BEGIN");
22750: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
22751: 
22752: 			$s = "INSERT INTO `task_list`
22753: 				SET 
22754: 					" . implode(",\n", $s) ."
22755: 				";
22756: 
22757: 			$q = query($s);
22758: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');
22759: 
22760: 			$id_val = $id_task_list = insert_id();
22761: 			$id_key = 'id_task_list';
22762: 
22763: 			foreach($add_table_list as $id_list)
22764: 			{
22765: 				if (!is_array($id_list[2])) return $this->showError('404', 'error', "{$id_list[3]} : wrong data");
22766: 				$s = array();
22767: 				$list_table = $id_list[0];
22768: 				if (empty($id_list[1])){
22769: 					if (empty($id_list[4])) continue;
22770: 					if (!empty($id_list[5]))
22771: 					{
22772: 						$s_d = "DELETE
22773: 							FROM `" . $list_table . "`
22774: 							WHERE 
22775: 								`" . $id_key . "` = '" . $id_val . "'
22776: 							";
22777: 						$q = query($s_d);
22778: 			
22779: 						if ($q === false) return $this->showError('404', 'error', "{$id_list[3]}: database delete failed");
22780: 					}
22781: 					$allowed_fields = $id_list[4];
22782: 					$forbidden_fields_inner = array();
22783: 					$affected_fields_inner = array();
22784: 					foreach ($id_list[2] as $j=>$d_arr)
22785: 					{	
22786: 						$filtered_data = array();
22787: 						$sql_if = array();
22788: 						foreach ($d_arr as $key => $value)
22789: 						{
22790: 							if (isset($allowed_fields[$key]))
22791: 							{
22792: 								$affected_fields_inner[$j][] = $key;
22793: 								if (!empty($allowed_fields[$key]['format']))
22794: 								{
22795: 									$value = $allowed_fields[$key]['format']($value,$id_val);
22796: 								}
22797: 								if (empty($value['error']))
22798: 								{
22799: 									$name = $allowed_fields[$key]['name'];
22800: 									$null_on = $allowed_fields[$key]['NULL'];
22801: 									if ($null_on === true || $value !== NULL)
22802: 									{
22803: 										$filtered_data[$name] = $value;
22804: 									}
22805: 									else
22806: 									{
22807: 										return $this->showError('404', 'error', "{$id_list[3]} $key: null value");
22808: 									}
22809: 								}
22810: 								else
22811: 								{
22812: 									return $this->showError('404', 'error', "{$id_list[3]} $j $key: {$value['error']}");
22813: 								}
22814: 							}
22815: 							else
22816: 							{
22817: 								$forbidden_fields_inner[$j][] = $key;
22818: 							}
22819: 						}
22820: 
22821: 						$s = array();
22822: 						foreach ($filtered_data as $key => $value)
22823: 						{
22824: 							$s[] = "`" . $key . "` = " 
22825: 								 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
22826: 						}
22827: 						$s = implode(",", $s);
22828: 						if (!empty($s)) $s = ", $s";
22829: 						$s = "`" . $id_key . "` = '" . $id_val . "'" . $s;
22830: 
```

### `createTrip` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `11365-12128`
```php
11365: 		public function createTrip($id_user = "", $data = "", $langs = array(), $s_data = array())
11366: 		{
11367: 			if (empty($_SESSION[UID])) {
11368: 				return $this->showError('404', 'error', 'unauthorized access');
11369: 			}
11370: 
11371: 			if (empty($id_user) || $id_user == $_SESSION[UID])
11372: 			{
11373: 				if ($this->id_role != 2)
11374: 				{
11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
11376: 				}
11377: 				else
11378: 				{
11379: 					if ($_SESSION['id_verification_status'] != 2)
11380: 					{
11381: 						return $this->showError('404', 'error', 'wrong user check state');
11382: 					}
11383: 					if (!empty($_SESSION['user_ban_status']['order']))
11384: 					{
11385: 						return $this->showError('404', 'error', 'user banned');
11386: 					}
11387: 				}
11388: 				$id_user = $_SESSION[UID];
11389: 			}
11390: 			else
11391: 			{
11392: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11393: 				$s = "SELECT 
11394: 						`id_user`
11395: 					FROM `users`
11396: 					WHERE 
11397: 						`id_user` = '" . $id_user . "'
11398: 					LIMIT 1
11399: 					";
11400: 
11401: 				$q = query($s);
11402: 				if ($q === false) return $this->showError('404', 'error', 'user select failed');
11403: 				$d = fetch_assoc($q);
11404: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');
11405: 			}
11406: 
11407: 			if (empty($data)) 
11408: 			{
11409: 				return $this->showError('404', 'error', 'empty data');
11410: 			}
11411: 
11412: 			$data = json_decode($data,true);
11413: 			
11414: 			if (empty($data) || !is_array($data)) 
11415: 			{
11416: 				return $this->showError('404', 'error', 'wrong data');
11417: 			}
11418: 			$iso = array();
11419: 			foreach ($langs as $lang)
11420: 			{
11421: 				$iso[$lang['iso']] = array(
11422: 										'name'				=>$lang['iso'],
11423: 										'type'				=>'string',
11424: 										'NULL'				=>false,
11425: 										'values'			=>NULL,
11426: 										'default'			=>NULL
11427: 				);
11428: 			}
11429: 			$options_valid_keys = array();
11430: 			foreach ($this->constant['t_options_valid_keys'] as $key => $value)
11431: 			{
11432: 				$options_valid_keys[$key] = array(
11433: 										'name'				=>$key,
11434: 										'type'				=>'string',
11435: 										'NULL'				=>false,
11436: 										'values'			=>NULL,
11437: 										'default'			=>NULL
11438: 				);
11439: 			}		
11440: 			$allowed_fields = array(						
11441: 				't_start_address'			=>		array(
11442: 							'name'				=>'from',
11443: 							'type'				=>'string',
11444: 							'NULL'				=>false,
11445: 							'values'			=>NULL,
11446: 							'default'			=>array(""),
11447: 							'error'				=>array(
11448: 								'requirement'     =>array(
11449: 									array('empty','from'),
11450: 									array('&&'),
11451: 									array(
11452: 										array('!isset','from_lat'),
11453: 										array('||'),
11454: 										array('!isset','from_lng'),
11455: 									)
11456: 								),
11457: 								'msg' 			=>'empty start address'
11458: 							)
11459: 				),
11460: 
11461: 				't_start_latitude'			=>		array(
11462: 							'name'				=>'from_lat',
11463: 							'type'				=>'latitude',
11464: 							'NULL'				=>true,
11465: 							'values'			=>NULL,
11466: 							'default'			=>array(NULL)
11467: 				),
11468: 				't_start_longitude'			=>		array(
11469: 							'name'				=>'from_lng',
11470: 							'type'				=>'longitude',
11471: 							'NULL'				=>true,
11472: 							'values'			=>NULL,
11473: 							'default'			=>array(NULL)
11474: 				),
11475: 				't_destination_address'		=>		array(
11476: 							'name'				=>'to',
11477: 							'type'				=>'string',
11478: 							'NULL'				=>false,
11479: 							'values'			=>NULL,
11480: 							'default'			=>array("")
11481: 				),
11482: 				't_destination_latitude'	=>		array(
11483: 							'name'				=>'to_lat',
11484: 							'type'				=>'latitude',
11485: 							'NULL'				=>true,
11486: 							'values'			=>NULL,
11487: 							'default'			=>array(NULL)
11488: 				),
11489: 				't_destination_longitude'	=>		array(
11490: 							'name'				=>'to_lng',
11491: 							'type'				=>'longitude',
11492: 							'NULL'				=>true,
11493: 							'values'			=>NULL,
11494: 							'default'			=>array(NULL)
11495: 				),				
11496: 				't_start_datetime_interval'	=>		array(
11497: 							'name'				=>'start_plan_datetime_interval',
11498: 							'type'				=>'second',
11499: 							'NULL'				=>false,
11500: 							'values'			=>NULL,
11501: 							'default'			=>array(0)
11502: 				),				
11503: 				't_start_datetime'			=>		array(
11504: 							'name'				=>'start_plan_datetime',
11505: 							'type'				=>'datetime',
11506: 							'NULL'				=>false,
11507: 							'values'			=>NULL,
11508: 							'default'			=>array('0000-00-00 00:00:00'),
11509: 							'error'				=>array(
11510: 								'requirement'     =>array(
11511: 									array('empty','start_plan_datetime')
11512: 								),
11513: 								'msg' 			=>'empty trip datetime'
11514: 							)
11515: 				),
11516: 				't_complete_datetime'		=>		array(
11517: 							'name'				=>'complete_plan_datetime',
11518: 							'type'				=>'datetime',
11519: 							'NULL'				=>false,
11520: 							'values'			=>NULL,
11521: 							'default'			=>array('0000-00-00 00:00:00')
11522: 				),
11523: 				't_options'					=>		array(
11524: 							'name'				=>'json',
11525: 							'type'				=>'associative for json',
11526: 							'NULL'				=>false,
11527: 							'values'			=>NULL,
11528: 							'default'			=>array(""),
11529: 							'keys'				=>$options_valid_keys ? : NULL
11530: 				),
11531: 				'price_time_function'					=>		array(
11532: 							'name'				=>'id_price_time_function',
11533: 							'type'				=>'integer',
11534: 							'NULL'				=>true,
11535: 							'values'			=>NULL,
11536: 							'default'			=>array(NULL)
11537: 				),
11538: 				'sc_id'					=>		array(
11539: 							'name'				=>'id_schedule',
11540: 							'type'				=>'integer',
11541: 							'NULL'				=>true,
11542: 							'values'			=>NULL,
11543: 							'default'			=>array(NULL)
11544: 				),
11545: 				'currency'				=>		array(
11546: 							'name'				=>'currency',
11547: 							'type'				=>'ISO 4217 alpha code',
11548: 							'NULL'				=>true,
11549: 							'values'			=>NULL,
11550: 							'default'			=>array(NULL)
11551: 				),
11552: 				'currency_priority'		=>		array(
11553: 							'name'				=>'currency_priority',
11554: 							'type'				=>'boolean integer',
11555: 							'NULL'				=>true,
11556: 							'values'			=>NULL,
11557: 							'default'			=>array(NULL)
11558: 				),
11559: 				'fee'					=>		array(
11560: 							'name'				=>'fee',
11561: 							'type'				=>'double',
11562: 							'NULL'				=>true,
11563: 							'values'			=>NULL,
11564: 							'default'			=>array(NULL)
11565: 				),
11566: 				'tariff'				=>		array(
11567: 							'name'				=>'tariff',
11568: 							'type'				=>'double',
11569: 							'NULL'				=>true,
11570: 							'values'			=>NULL,
11571: 							'default'			=>array(NULL)
11572: 				),
11573: 				'tariff_priority'		=>		array(
11574: 							'name'				=>'tariff_priority',
11575: 							'type'				=>'boolean integer',
11576: 							'NULL'				=>true,
11577: 							'values'			=>NULL,
11578: 							'default'			=>array(NULL)
11579: 				),
11580: 				'ag_id'					=>		array(
11581: 							'name'				=>'id_aggregator',
11582: 							'type'				=>'integer',
11583: 							'NULL'				=>true,
11584: 							'values'			=>NULL,
11585: 							'default'			=>array(NULL)
```

### `deleteDropboxFile` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `23356-23431`
```php
23356: 		public function deleteDropboxFile($id_dropbox_link = '')
23357: 		{
23358: 			if (empty($_SESSION[UID])) {
23359: 				return $this->showError('404', 'error', 'unauthorized access');
23360: 			}
23361: 			if (empty($id_dropbox_link)) 
23362: 			{
23363: 				return $this->showError('404', 'error', 'empty dl_id');
23364: 			}
23365: 
23366: 			$s = "SELECT
23367: 					`id_dropbox_link`,
23368: 					`json`,
23369: 					`deleted`,
23370: 					IF(`deleted` = 0,(
23371: 						SELECT
23372: 							`id_user`
23373: 						FROM
23374: 							`users_dropbox_link`
23375: 						WHERE
23376: 							`users_dropbox_link`.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link` AND `owner` = '1'
23377: 					),'') as user
23378: 				FROM `dropbox_link` 				
23379: 				WHERE	
23380: 					`id_dropbox_link` = '" . $id_dropbox_link  . "'
23381: 				LIMIT 1
23382: 				";
23383: 
23384: 			$q = query($s);
23385: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
23386: 
23387: 			$d = fetch_assoc($q);
23388: 			if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link not found');
23389: 			if (!empty($d['deleted'])) return $this->showError('404', 'error', 'dropbox file already deleted');
23390: 			if ($this->id_role != 4 && $d['user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
23391: 			$json = json_decode($d['json'],true);
23392: 			
23393: 			$response = delete_from_dropbox($id_dropbox_link);
23394: 			if (!empty($response['error'])) return $this->showError('404', 'error', $response['error']);
23395: 			$json['response_del'] = $response['data'];
23396: 			$json['owner'] = $d['user'];
23397: 
23398: 			$s = "DELETE
23399: 				FROM `users_dropbox_link`
23400: 				WHERE 
23401: 					`id_dropbox_link` = ' $id_dropbox_link'
23402: 				";
23403: 			$q = query($s);
23404: 
23405: 			if ($q === false) return $this->showError('404', 'error', 'delete failed');
23406: 			
23407: 			$s = "UPDATE `dropbox_link`
23408: 				SET
23409: 					`json` = '" . real_escape_string(json_encode($json)) . "',
23410: 					`deleted` = 1
23411: 				WHERE
23412: 					`id_dropbox_link` = '" . $id_dropbox_link . "'
23413: 				";
23414: 
23415: 			$q = query($s);
23416: 
23417: 			$qs = affected_rows();
23418: 			if ($qs === -1) 
23419: 			{
23420: 				$warning = 'update query failed';
23421: 			}
23422: 			elseif ($qs === 0) 
23423: 			{
23424: 				$warning = 'modified data not found for query';
23425: 			}
23426: 
23427: 			return array(
23428: 				'code' 		=>	'200',
23429: 				'status' 	=>	'success'
23430: 			);
23431: 		}
```

### `depositCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `25191-25448`
```php
25191: 		public function depositCurrencyAccount($data = '', $appUrl = '', $payment_services = array(), $payment_ways = array(), $account_currency_list = array())
25192: 		{
25193: 			if (empty($_SESSION[UID])) {
25194: 				return $this->showError('404', 'error', 'unauthorized access');
25195: 			}
25196: 
25197: 			if (empty($data)) 
25198: 			{
25199: 				return $this->showError('404', 'error', 'empty data');
25200: 			}
25201: 
25202: 			$data = json_decode($data,true);
25203: 			
25204: 			if (empty($data) || !is_array($data)) 
25205: 			{
25206: 				return $this->showError('404', 'error', 'wrong data');
25207: 			}
25208: 
25209: 			$allowed_fields = array(
25210: 				
25211: 				'a_id'	=>		array(
25212: 										'name'	=>	false,
25213: 										'desc'	=>	'id_currency_account'
25214: 									),			
25215: 				'sum'	=>		array(
25216: 										'name'	=>	'sum',
25217: 										'NULL'	=>	true
25218: 									),
25219: 				'currency'	=>		array(
25220: 										'name'	=>	'currency',
25221: 										'NULL'	=>	true
25222: 									),
25223: 									
25224: 				'payment_service'	=>		array(
25225: 										'name'	=>	false,
25226: 										'desc'	=>	'id_payment_service'
25227: 									),
25228: 				'payment_way'	=>		array(
25229: 										'name'	=>	false,
25230: 										'desc'	=>	'id_payment_method'
25231: 									),
25232: 				'json'	=>		array(
25233: 										'name'	=>	'json',
25234: 										'NULL'	=>	false
25235: 									)
25236: 			);
25237: 
25238: 			$forbidden_fields = array();
25239: 			$affected_fields = array();
25240: 			$filtered_data = array();
25241: 			$add_table_list = array();
25242: 			foreach ($data as $key => $value)
25243: 			{
25244: 				if (isset($allowed_fields[$key]))
25245: 				{
25246: 					if (!empty($allowed_fields[$key]['format']))
25247: 					{
25248: 						$value = $allowed_fields[$key]['format']($value);
25249: 					}
25250: 					if (empty($value['error']))
25251: 					{
25252: 						if ($allowed_fields[$key]['name'] === NULL)
25253: 						{
25254: 							$add_table_list[] = array(
25255: 								$allowed_fields[$key]['table'],
25256: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
25257: 								$value,
25258: 								$key,
25259: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
25260: 								empty($data['exact']) ? false : true
25261: 							);
25262: 						}
25263: 						elseif (isset($allowed_fields[$key]['desc']))
25264: 						{
25265: 							continue;
25266: 						}
25267: 						else
25268: 						{									
25269: 							$name = $allowed_fields[$key]['name'];
25270: 							$null_on = $allowed_fields[$key]['NULL'];
25271: 							if ($null_on === true || $value !== NULL)
25272: 							{
25273: 								$affected_fields[] = $key;
25274: 								$filtered_data[$name] = $value;
25275: 							}
25276: 							else
25277: 							{
25278: 								return $this->showError('404', 'error', "$key: null value");
25279: 							}
25280: 						}
25281: 					}
25282: 					else
25283: 					{
25284: 						return $this->showError('404', 'error', "$key: {$value['error']}");
25285: 					}
25286: 				}
25287: 				else
25288: 				{
25289: 					$forbidden_fields[] = $key;
25290: 				}
25291: 			}
25292: 
25293: 			if (empty($filtered_data['sum'])) return $this->showError('404', 'error', 'empty sum');
25294: 			if (empty($filtered_data['currency'])) $filtered_data['currency'] = DEFAULT_CURRENCY;
25295: 			if (!isset($account_currency_list[$filtered_data['currency']])) return $this->showError('404', 'error', 'wrong currency');
25296: 
25297: 			if (empty($data['payment_service'])) $data['payment_service'] = 1;
25298: 			if (empty($payment_services[$data['payment_service']])) return $this->showError('404', 'error', 'wrong payment service');
25299: 			if (empty($payment_services[$data['payment_service']]['var'])) return $this->showError('404', 'error', 'empty payment service var');
25300: 			
25301: 			$f_name = "{$payment_services[$data['payment_service']]['var']}_create_payment";		
25302: 			if (!function_exists($f_name)) return $this->showError('404', 'error', 'function not found');
25303: 			if (empty($data['payment_way'])) $data['payment_way'] = 2;
25304: 			if (empty($payment_ways[$data['payment_way']])) return $this->showError('404', 'error', 'wrong payment way');			
25305: 			foreach($payment_services[$data['payment_service']]['payment_ways'] as $key=>$val)
25306: 			{
25307: 				if ($val[0] == $data['payment_way']) 
25308: 				{
25309: 					$p_w_index = $key;	
25310: 					if (!empty($val[1]['api']['type'])) $p_w_type =  $val[1]['api']['type'];
25311: 				}
25312: 			}
25313: 			if (!isset($p_w_index)) return $this->showError('404', 'error', 'forbidden payment way');
25314: 			if (!isset($p_w_type)) return $this->showError('404', 'error', 'empty payment way type');
25315: 
25316: 			$q = query("BEGIN");
25317: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
25318: 
25319: 			$out = array('affected_fields' 	=> 	$affected_fields);
25320: 			if (empty($data['a_id'])) 
25321: 			{
25322: 				$s = "SELECT			
25323: 					  `id_currency_account`,
25324: 					  `id_user`,
25325: 					  `currency`,
25326: 					  `id_currency_account_status`
25327: 					FROM `currency_account`
25328: 					WHERE	
25329: 						`id_user` = '{$_SESSION[UID]}' AND `currency` = '{$filtered_data['currency']}'
25330: 					LIMIT 1
25331: 					FOR UPDATE
25332: 					";
25333: 
25334: 				$q = query($s);
25335: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
25336: 				$d = fetch_assoc($q);
25337: 				if (empty($d['id_currency_account']))
25338: 				{
25339: 					$now_time = time();
25340: 					$s = "INSERT INTO `currency_account`
25341: 						SET 
25342: 						  `id_user` = '{$_SESSION[UID]}',
25343: 						  `sum`= 0,
25344: 						  `currency` = '{$filtered_data['currency']}',
25345: 						  `reserved` = 0,
25346: 						  `create_user` = '{$_SESSION[UID]}',
25347: 						  `last_edit_user` = '{$_SESSION[UID]}',
25348: 						  `last_edit_int_timestamp` = '$now_time',
25349: 						  `create_int_timestamp` = '$now_time'
25350: 						";
25351: 
25352: 					$q = query($s);
25353: 					if ($q === false) return $this->showError('404', 'error', 'insert failed');
25354: 					$a_id = insert_id();
25355: 					$out['a_id'] = $a_id;
25356: 				}
25357: 				else
25358: 				{
25359: 					if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
25360: 					$a_id = $d['id_currency_account'];
25361: 				}
25362: 			}
25363: 			else
25364: 			{
25365: 				$s = "SELECT			
25366: 					  `id_currency_account`,
25367: 					  `id_user`,
25368: 					  `currency`,
25369: 					  `id_currency_account_status`
25370: 					FROM `currency_account`
25371: 					WHERE	
25372: 						`id_currency_account` = '{$data['a_id']}'
25373: 					LIMIT 1
25374: 					FOR UPDATE
25375: 					";
25376: 
25377: 				$q = query($s);
25378: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
25379: 				$d = fetch_assoc($q);
25380: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'account not found');
25381: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25382: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
25383: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
25384: 				$a_id = $data['a_id'];
25385: 			}
25386: 			
25387: 			$s = array();
25388: 			foreach ($filtered_data as $key => $value)
25389: 			{
25390: 				$s[] = "`$key` = " 
25391: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
25392: 			}
25393: 
25394: 			$total_sum = ((float)$filtered_data['sum']) * (1  - ($this->constant['commission_deposit']) / 100);
25395: 			$now_time = time();
25396: 			$s = "INSERT INTO `transaction`
25397: 				SET 
25398: 				  " . implode(",\n", $s) .",
25399: 				  `total_sum` = '$total_sum',
25400: 				  `to` = '$a_id',
25401: 				  `commission` = '{$this->constant['commission_deposit']}',
25402: 				  `id_transaction_type` = '1',
25403: 				  `create_user` = '{$_SESSION[UID]}',
25404: 				  `last_edit_user` = '{$_SESSION[UID]}',
25405: 				  `last_edit_int_timestamp` = '$now_time',
25406: 				  `create_int_timestamp` = '$now_time'
25407: 				";
25408: 
25409: 			$q = query($s);
25410: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
25411: 			
```

### `editContact` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `21309-21539`
```php
21309: 		public function editContact($data = '', $id_contact_item = '', $langs = array())
21310: 		{	
21311: 			if (empty($_SESSION[UID])) {
21312: 				return $this->showError('404', 'error', 'unauthorized access');
21313: 			}
21314: 
21315: 			if (empty($id_contact_item)) return $this->showError('404', 'error', 'empty co_id');
21316: 
21317: 			if (empty($data)) 
21318: 			{
21319: 				return $this->showError('404', 'error', 'empty data');
21320: 			}
21321: 
21322: 			$data = json_decode($data,true);
21323: 			
21324: 			if (empty($data) || !is_array($data)) 
21325: 			{
21326: 				return $this->showError('404', 'error', 'wrong data');
21327: 			}
21328: 
21329: 			$allowed_fields = array(
21330: 				'owner'	=>		array(
21331: 										'name'	=>	'owner',
21332: 										'NULL'	=>	false
21333: 									),
21334: 				'o_type'	=>		array(
21335: 										'name'	=>	'id_owner_type',
21336: 										'NULL'	=>	false
21337: 									),
21338: 				'co_class'	=>		array(
21339: 										'name'	=>	'id_contact_type',
21340: 										'NULL'	=>	false
21341: 									),
21342: 				'number'	=>	array(
21343: 										'name'	=>	'contact_number',
21344: 										'NULL'	=>	true
21345: 									),
21346: 				'cid'	=>		array(
21347: 										'name'	=>	'contact_id',
21348: 										'NULL'	=>	true
21349: 									),
21350: 				'link'	=>		array(
21351: 										'name'	=>	'contact_link',
21352: 										'NULL'	=>	true
21353: 									),
21354: 				'is_bot'	=>		array(
21355: 										'name'	=>	'is_bot',
21356: 										'NULL'	=>	false,
21357: 										'format'=>	function($val){return empty($val) ? 0 : 1;}
21358: 									),
21359: 				'key1'	=>		array(
21360: 										'name'	=>	'key1',
21361: 										'NULL'	=>	true
21362: 									),
21363: 				'key2'	=>		array(
21364: 										'name'	=>	'key2',
21365: 										'NULL'	=>	true
21366: 									),
21367: 				'token'	=>		array(
21368: 										'name'	=>	'token',
21369: 										'NULL'	=>	true
21370: 									),
21371: 				'hash'	=>		array(
21372: 										'name'	=>	'hash',
21373: 										'NULL'	=>	true
21374: 									),
21375: 				'secret'	=>		array(
21376: 										'name'	=>	'secret',
21377: 										'NULL'	=>	true
21378: 									),
21379: 				'host'	=>		array(
21380: 										'name'	=>	'host',
21381: 										'NULL'	=>	true
21382: 									),	
21383: 				'port'	=>		array(
21384: 										'name'	=>	'port',
21385: 										'NULL'	=>	true
21386: 									),
21387: 				'login'	=>		array(
21388: 										'name'	=>	'login',
21389: 										'NULL'	=>	true
21390: 									),
21391: 				'password'	=>		array(
21392: 										'name'	=>	'password',
21393: 										'NULL'	=>	true
21394: 									),
21395: 				'smtpsecure'	=>		array(
21396: 										'name'	=>	'smtpsecure',
21397: 										'NULL'	=>	true
21398: 									),
21399: 				'fromname'	=>		array(
21400: 										'name'	=>	'fromname',
21401: 										'NULL'	=>	true
21402: 									),
21403: 				'active'	=>		array(
21404: 										'name'	=>	'active',
21405: 										'NULL'	=>	false,
21406: 										'format'=>	function($val){return empty($val) ? 0 : 1;}
21407: 									)
21408: 			);
21409: 			foreach ($langs as $lang)
21410: 			{
21411: 				$out_key = $lang['iso'];
21412: 				$db_key = 'name_' . $lang['iso'];
21413: 				$null_on = true;
21414: 				$allowed_fields[$out_key] = array(
21415: 									'name'	=>	$db_key ,
21416: 									'NULL'	=>	$null_on
21417: 				);				
21418: 				$out_key = 'about_' . $lang['iso'];
21419: 				$db_key = 'description_' . $lang['iso'];
21420: 				$null_on = false;
21421: 				$allowed_fields[$out_key] = array(
21422: 									'name'	=>	$db_key ,
21423: 									'NULL'	=>	$null_on
21424: 				);
21425: 			}
21426: 
21427: 			$q = query("BEGIN");
21428: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
21429: 
21430: 			$s = "SELECT			
21431: 					`contact_items`.`id_contact_item`,
21432: 					`contact_items`.`owner`,
21433: 					`contact_items`.`id_owner_type`
21434: 				FROM `contact_items`
21435: 				WHERE	
21436: 					`id_contact_item` = '$id_contact_item'
21437: 				LIMIT 1
21438: 				FOR UPDATE
21439: 				";
21440: 
21441: 			$q = query($s);
21442: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
21443: 
21444: 			$d = fetch_assoc($q);
21445: 			if (empty($d['id_contact_item'])) return $this->showError('404', 'error', 'contact not found');
21446: 			if ($this->id_role != 4 && ($d['id_owner_type'] != 1 || $d['owner'] != $_SESSION[UID])) return $this->showError('404', 'error', 'not enough rights');
21447: 
21448: 			if ($this->id_role != 4)
21449: 			{
21450: 				unset($allowed_fields['owner']);
21451: 				unset($allowed_fields['o_type']);
21452: 			}
21453: 			$forbidden_fields = array();
21454: 			$affected_fields = array();
21455: 			$filtered_data = array();
21456: 			foreach ($data as $key => $value)
21457: 			{
21458: 				if (isset($allowed_fields[$key]))
21459: 				{
21460: 					$affected_fields[] = $key;
21461: 					if (!empty($allowed_fields[$key]['format']))
21462: 					{
21463: 						$value = $allowed_fields[$key]['format']($value);
21464: 					}
21465: 					if (empty($value['error']))
21466: 					{
21467: 						$name = $allowed_fields[$key]['name'];
21468: 						$null_on = $allowed_fields[$key]['NULL'];
21469: 						if ($null_on === true || $value !== NULL)
21470: 						{
21471: 							$filtered_data[$name] = $value;
21472: 						}
21473: 						else
21474: 						{
21475: 							return $this->showError('404', 'error', "$key: null value");
21476: 						}
21477: 					}
21478: 					else
21479: 					{
21480: 						return $this->showError('404', 'error', "$key: {$value['error']}");
21481: 					}
21482: 				}
21483: 				else
21484: 				{
21485: 					$forbidden_fields[] = $key;
21486: 				}
21487: 			}
21488: 
21489: 			$s = array();
21490: 			foreach ($filtered_data as $key => $value)
21491: 			{
21492: 				$s[] = "`$key` = " 
21493: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
21494: 			}
21495: 
21496: 			if (empty($s)) return $this->showError('404', 'error', 'allowed data not found');
21497: 	
21498: 			$s = "UPDATE `contact_items`
21499: 				SET 
21500: 					" . implode(",\n", $s) ."
21501: 				WHERE
21502: 					`id_contact_item` = '$id_contact_item'
21503: 				";
21504: 
21505: 			query($s);
21506: 	
21507: 			global $link;
21508: 			if (mysqli_affected_rows($link) === -1) 
21509: 			{
21510: 				return $this->showError('404', 'error', 'database update failed');
21511: 			}
21512: 			elseif (mysqli_affected_rows($link) === 0) 
21513: 			{
21514: 				return $this->showError('404', 'error', 'modified data not found');
21515: 			}
21516: 
21517: 			$s = "UPDATE `contact_items`
21518: 				SET
21519: 					`last_edit_datetime` = now(),
21520: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
21521: 				WHERE
21522: 					`id_contact_item` = '$id_contact_item'
21523: 				";
21524: 
21525: 			$q = query($s);
21526: 			if ($q === false) return $this->showError('404', 'error', 'database timestamp update failed');
21527: 			
21528: 			$q = query("COMMIT");
21529: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
```

### `editData` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `14340-14840`
```php
14340: 		public function editData($data = 0, $s_date = array(), $s_date_private = array(), $s_data_stt = array(), $s_data_sc = array())
14341: 		{
14342: 			if (empty($_SESSION[UID])) {
14343: 				return $this->showError('404', 'error', 'unauthorized access');
14344: 			}
14345: 
14346: 			if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
14347: 			{
14348: 				return $this->showError('404', 'error', 'wrong user check state');
14349: 			}
14350: 			
14351: 			if (empty($data)) 
14352: 			{
14353: 				return $this->showError('404', 'error', 'empty data');
14354: 			}
14355: 
14356: 			$data = json_decode($data,true);
14357: 			
14358: 			if (empty($data) || !is_array($data)) 
14359: 			{
14360: 				return $this->showError('404', 'error', 'wrong data');
14361: 			}
14362: 
14363: 			$data_template = array(
14364: 				'langs'	=>	array(
14365: 					'table'			=>	'lang',
14366: 					'id'			=>	'id_lang',
14367: 					'data_suffix'	=>	'',
14368: 					'allowed_fields'=>	array(
14369: 						'native'	=>		array(
14370: 												'name'	=>	'native_name',
14371: 												'NULL'	=>	false,
14372: 											),
14373: 						'iso'		=>		array(
14374: 												'name'	=>	function($val){if($val === NULL)return 'ISO 639-1';if(strlen($val) === 2)return 'ISO 639-1';return 'ISO 639-3';},
14375: 												'NULL'	=>	true
14376: 											),
14377: 						'logo'		=>		array(
14378: 												'name'	=>	'logo_link',
14379: 												'NULL'	=>	false,
14380: 											),
14381: 						'active'	=>		array(
14382: 												'name'	=>	'active',
14383: 												'NULL'	=>	false,
14384: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14385: 											)
14386: 					)
14387: 				
14388: 				),
14389: 				'lang_vls'	=>	array(
14390: 					'table'			=>	'lang_values',
14391: 					'key'			=>	array('name','id_lang','value'),
14392: 					'data_suffix'	=>	''
14393: 				),
14394: 				'accounts'	=>	array(
14395: 					'table'			=>	'account',
14396: 					'id'			=>	'id_account',
14397: 					'data_suffix'	=>	'',
14398: 					'allowed_fields'=>	array(
14399: 						'date_place'	=>		array(
14400: 												'name'	=>	'id_date_place',
14401: 												'NULL'	=>	true
14402: 											),										
14403: 						'login'	=>		array(
14404: 												'name'	=>	'login',
14405: 												'NULL'	=>	true
14406: 											),
14407: 						'email'	=>		array(
14408: 												'name'	=>	'email',
14409: 												'NULL'	=>	true
14410: 											),
14411: 						'phone'	=>		array(
14412: 												'name'	=>	'phone',
14413: 												'NULL'	=>	true
14414: 											),
14415: 						'p_word'	=>		array(
14416: 												'name'	=>	'pwd',
14417: 												'NULL'	=>	true
14418: 											),
14419: 						'auth'	=>		array(
14420: 												'name'	=>	'auth',
14421: 												'NULL'	=>	false,
14422: 												'format'=>	function($val){return empty($val) ? 0 : ((int)$val < 0 ? - 1 : 1);}
14423: 											),
14424: 						'json'	=>		array(
14425: 												'name'	=>	'json',
14426: 												'NULL'	=>	false
14427: 											),
14428: 						'active'	=>		array(
14429: 												'name'	=>	'active',
14430: 												'NULL'	=>	false,
14431: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14432: 											)
14433: 					)
14434: 				),
14435: 				'address_objects'	=>	array(
14436: 					'table'			=>	'address_object',
14437: 					'id'			=>	'id_address_object_type',
14438: 					'data_suffix'	=>	'',					
14439: 					'auto_fields'	=>	array(
14440: 						'timestamp_edited' => 'last_edit_int_timestamp',
14441: 						'timestamp_created' => 'create_int_timestamp',
14442: 						'e_u_id' => 'last_edit_user',
14443: 						'c_u_id' => 'create_user'
14444: 					),
14445: 					'allowed_fields'=>	array(
14446: 						'upper'	=>		array(
14447: 												'name'	=>	'id_address_object_upper',
14448: 												'NULL'	=>	true
14449: 											),
14450: 						'type'	=>		array(
14451: 												'name'	=>	'id_address_object_type',
14452: 												'NULL'	=>	true
14453: 											),
14454: 						'json'	=>		array(
14455: 												'name'	=>	'json',
14456: 												'NULL'	=>	false
14457: 											),
14458: 											
14459: 						'country'	=>		array(
14460: 												'name'	=>	'country',
14461: 												'NULL'	=>	true
14462: 											),
14463: 						'region'	=>		array(
14464: 												'name'	=>	'id_region',
14465: 												'NULL'	=>	true
14466: 											),
14467: 						'city'	=>		array(
14468: 												'name'	=>	'id_city',
14469: 												'NULL'	=>	true
14470: 											),
14471: 						'city_area'	=>		array(
14472: 												'name'	=>	'id_city_area',
14473: 												'NULL'	=>	true
14474: 											),
14475: 						'active'	=>		array(
14476: 												'name'	=>	'active',
14477: 												'NULL'	=>	false,
14478: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14479: 											)
14480: 					)
14481: 				),
14482: 				'address_object_types'	=>	array(
14483: 					'table'			=>	'address_object_type',
14484: 					'id'			=>	'id_address_object_type',
14485: 					'data_suffix'	=>	'',					
14486: 					'auto_fields'	=>	array(
14487: 						'timestamp_edited' => 'last_edit_int_timestamp',
14488: 						'timestamp_created' => 'create_int_timestamp',
14489: 						'e_u_id' => 'last_edit_user',
14490: 						'c_u_id' => 'create_user'
14491: 					),
14492: 					'allowed_fields'=>	array(
14493: 						'json'	=>		array(
14494: 												'name'	=>	'json',
14495: 												'NULL'	=>	false
14496: 											),
14497: 						'active'	=>		array(
14498: 												'name'	=>	'active',
14499: 												'NULL'	=>	false,
14500: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14501: 											)
14502: 					)
14503: 				),
14504: 				'aggregators'	=>	array(
14505: 					'table'			=>	'aggregator',
14506: 					'id'			=>	'id_aggregator',
14507: 					'data_suffix'	=>	'',
14508: 					'allowed_fields'=>	array(
14509: 						'json'	=>		array(
14510: 												'name'	=>	'json',
14511: 												'NULL'	=>	false
14512: 											),
14513: 						'active'	=>		array(
14514: 												'name'	=>	'active',
14515: 												'NULL'	=>	false,
14516: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14517: 											)
14518: 					)
14519: 				),
14520: 				'car_bodies'	=>	array(
14521: 					'table'			=>	'car_body',
14522: 					'id'			=>	'id_car_body',
14523: 					'data_suffix'	=>	'',
14524: 					'allowed_fields'=>	array(
14525: 						'active'	=>		array(
14526: 												'name'	=>	'active',
14527: 												'NULL'	=>	false,
14528: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14529: 											)
14530: 					)
14531: 				),
14532: 				'car_classes'	=>	array(
14533: 					'table'			=>	'car_class',
14534: 					'id'			=>	'id_car_class',
14535: 					'data_suffix'	=>	'',
14536: 					'allowed_fields'=>	array(
14537: 						'seats'		=>		array(
14538: 												'name'	=>	'seats',
14539: 												'NULL'	=>	false
14540: 											),
14541: 						'luggage'		=>		array(
14542: 												'name'	=>	'luggage',
14543: 												'NULL'	=>	false
14544: 											),
14545: 						'photo'		=>	array(
14546: 												'name'	=>	'photo_link',
14547: 												'NULL'	=>	false,
14548: 												'format'=>	function($val){
14549: 																list($type, $image) = array_merge(explode(';', $val),array(''));
14550: 																list(, $image) = array_merge(explode(',', $image),array(''));
14551: 																$image = base64_decode($image);
14552: 																if (!empty($image))
14553: 																{
14554: 																	$ext = mimeToext($type);
14555: 																	if (empty($ext)) return array('error' => 'wrong image type');				
14556: 																	while (file_exists($file = $_SERVER['DOCUMENT_ROOT'] . FILE_PATH . ($name_file = md5(md5(rand() . " " . rand())) . "." . $ext)));
14557: 																	@$file = file_put_contents($file, $image);
14558: 																	if (!$file) return array('error' => 'file not created');
14559: 																}
14560: 																else
```

### `editOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `9250-9750`
```php
9250: 		public function editOrder($data = "", $id_order = "")
9251: 		{
9252: 			if (empty($_SESSION[UID])) {
9253: 				return $this->showError('404', 'error', 'unauthorized access');
9254: 			}
9255: 
9256: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
9257: 			{
9258: 				return $this->showError('404', 'error', 'wrong user role');
9259: 			}
9260: 
9261: 			if (empty($data)) 
9262: 			{
9263: 				return $this->showError('404', 'error', 'empty data');
9264: 			}
9265: 
9266: 			$data = json_decode($data,true);
9267: 			
9268: 			if (empty($data) || !is_array($data)) 
9269: 			{
9270: 				return $this->showError('404', 'error', 'wrong data');
9271: 			}
9272: 
9273: 			$allowed_fields = array(
9274: 									'b_start_address'			=>		'from',
9275: 									'b_start_latitude'			=>		'from_lat',
9276: 									'b_start_longitude'			=>		'from_lng',
9277: 									'b_destination_address'		=>		'to',
9278: 									'b_destination_latitude'	=>		'to_lat',
9279: 									'b_destination_longitude'	=>		'to_lng',
9280: 									'b_start_datetime'			=>		'datetime_start_plan',
9281: 									'b_custom_comment'			=>		'comment',
9282: 									'b_flight_number'			=>		'flight_number',
9283: 									'b_terminal'				=>		'terminal',
9284: 									'b_passengers_count'		=>		'passenger_count',
9285: 									'b_luggage_count'			=>		'luggage_count',
9286: 									'b_placard'					=>		'placard',
9287: 									'b_car_class'				=>		'id_car_class',
9288: 									'b_payment_way'				=>		'id_payment_method',
9289: 									'b_payment_card'			=>		'id_payment_card',
9290: 									'b_tips'					=>		'tips',
9291: 									'b_cars_count'				=>		'car_count',
9292: //									'b_max_waiting'				=>		true,
9293: 									'b_options'					=>		'options',
9294: //									'b_comments'				=>		true,
9295: //									'b_services'				=>		true,
9296: 									'b_contact'					=>		'contact'
9297: 
9298: 
9299: 
9300: 
9301: 
9302: 			);
9303: 		
9304: 			$forbidden_fields = array();
9305: 			$affected_fields = array();
9306: 			$affected_keys = array();
9307: 			$filtered_data = array();
9308: 			$wrong_data_fields = array();
9309: 			
9310: 			$user_service_profile = NULL;
9311: 
9312: 			if ($this->id_role == 1 || $this->id_role == 5)
9313: 			{
9314: 
9315: 				$s = "SELECT
9316: 						`order`.`id_order`,
9317: 						`order`.`client`,
9318: 						`order`.`id_order_status`,
9319: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (5,6),1,NULL)) as started,
9320: 						`order`.`night_time`,
9321: 						`order`.`distance_estimate`,
9322: 						`order`.`id_car_class`,
9323: 						`order`.`from_lat`,
9324: 						`order`.`from_lng`,
9325: 						`order`.`to_lat`,
9326: 						`order`.`to_lng`,
9327: 						`order`.`car_count`,
9328: 						`order`.`datetime_start_plan`,
9329: 						GROUP_CONCAT(`order_driver`.`id_car`) as cars,
9330: 						`order`.`options`
9331: 					FROM `order` 
9332: 					LEFT JOIN `order_driver` USING (`id_order`)	
9333: 					WHERE	
9334: 						`id_order` = '" . $id_order . "'
9335: 					LIMIT 1
9336: 					FOR UPDATE
9337: 					";
9338: 
9339: 				$q = query($s);
9340: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
9341: 
9342: 				$d = fetch_assoc($q);
9343: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
9344: 				if ($d['client'] != $_SESSION[UID]) 
9345: 				{
9346: 					return $this->showError('404', 'error', 'user is not author');
9347: 				}
9348: 	
9349: 				$s = "SELECT 
9350: 						`value`
9351: 					FROM `users_prop_items_varchar`
9352: 					WHERE 
9353: 						`id_user` = '" . $_SESSION[UID] . "' AND `id_users_prop` = '3'
9354: 					LIMIT 1
9355: 					";
9356: 
9357: 				$q = query($s);
9358: 				if ($q === false) return $this->showError('404', 'error', 'user select failed');
9359: 				
9360: 				$d_u = fetch_assoc($q);
9361: 				if (isset($d_u['value'])) $user_service_profile = $d_u['value'];
9362: 
9363: 				$d['options'] = json_decode($d['options'],true);
9364: 
9365: 				switch ($d['id_order_status']) {
9366: 					case '1':
9367: 						$allowed_fields = array(
9368: 												'b_start_address'			=>		'from',				
9369: 												'b_start_latitude'			=>		'from_lat',
9370: 												'b_start_longitude'			=>		'from_lng',
9371: 												'b_destination_address'		=>		'to',
9372: 												'b_destination_latitude'	=>		'to_lat',
9373: 												'b_destination_longitude'	=>		'to_lng',
9374: 												'b_options'					=>		'options'
9375: 						);
9376: 						break;
9377: 					case '2':
9378: 						if ($user_service_profile == 'auction')
9379: 						{
9380: 							return $this->showError('404', 'error', "{$d['id_order_status']} is wrong booking state");
9381: 						
9382: 						}
9383: 						$allowed_fields = array(
9384: 												'b_destination_address'		=>		'to',
9385: 												'b_destination_latitude'	=>		'to_lat',
9386: 												'b_destination_longitude'	=>		'to_lng',
9387: 												'b_options'					=>		'options'
9388: 
9389: 						);
9390: 						if (empty($d['started']))
9391: 						{
9392: 							$allowed_fields['b_start_address'] = 'from';
9393: 							$allowed_fields['b_start_latitude'] = 'from_lat';
9394: 							$allowed_fields['b_start_longitude'] = 'from_lng';
9395: 						}
9396: 						break;
9397: 					case '5':
9398: 						$allowed_fields = array(
9399: 												'b_start_address'			=>		'from',						
9400: 												'b_start_latitude'			=>		'from_lat',
9401: 												'b_start_longitude'			=>		'from_lng',
9402: 												'b_destination_address'		=>		'to',
9403: 												'b_destination_latitude'	=>		'to_lat',
9404: 												'b_destination_longitude'	=>		'to_lng',
9405: 												'b_options'					=>		'options'
9406: 						);
9407: 						break;
9408: 					case '6':
9409: 						$allowed_fields = array(
9410: 												'b_start_address'			=>		'from',						
9411: 												'b_start_latitude'			=>		'from_lat',
9412: 												'b_start_longitude'			=>		'from_lng',
9413: 												'b_destination_address'		=>		'to',
9414: 												'b_destination_latitude'	=>		'to_lat',
9415: 												'b_destination_longitude'	=>		'to_lng',
9416: 												'b_options'					=>		'options'
9417: 						);
9418: 						break;
9419: 					default:
9420: 						return $this->showError('404', 'error', 'wrong booking state');
9421: 				}
9422: 
9423: 				foreach ($data as $key => $value)
9424: 				{
9425: 					if (!empty($allowed_fields[$key]))
9426: 					{
9427: 						if (is_string($value)) $data[$key] = trim($value);
9428: 						$affected_fields[] = $key;
9429: 						$affected_keys[$key] = true;
9430: 						if ($allowed_fields[$key] !== true) $filtered_data[$allowed_fields[$key]] = $data[$key];
9431: 					}
9432: 					else
9433: 					{
9434: 						$forbidden_fields[] = $key;
9435: 					}
9436: 				}
9437: 				
9438: 				if (empty($affected_fields)) return $this->showError('404', 'error', 'allowed data not found');
9439: 				
9440: 				if (!empty($affected_keys['b_options']))
9441: 				{
9442: 					if (empty($this->constant['b_options_edit_list_readonly']))
9443: 					{
9444: 						unset($this->constant['b_options_edit_list_keys'][':public']);
9445: 						unset($this->constant['b_options_edit_list_keys'][':u_id_alias']);
9446: 						if (empty($this->constant['b_options_edit_list_keys'])) 
9447: 						{
9448: 							$this->constant['b_options_edit_list_readonly'] = 1;
9449: 							$this->constant['b_options_edit_list_keys'][':public'] = true;
9450: 							$this->constant['b_options_edit_list_keys'][':u_id_alias'] = true;
9451: 						}
9452: 					}
9453: 					else
9454: 					{
9455: 						$this->constant['b_options_edit_list_keys'][':public'] = true;
9456: 						$this->constant['b_options_edit_list_keys'][':u_id_alias'] = true;
9457: 					}
9458: 					$options = $d['options'];
9459: 					if (!is_array($filtered_data['options'])) return $this->showError('404', 'error', 'b_options not array');
9460: 					$options_is_updated = false;
9461: 					foreach($filtered_data['options'] as $options_element)
9462: 					{
9463: 						if (!is_array($options_element)) return $this->showError('404', 'error', 'b_options element not array');
9464: 						if (empty($options_element[0]))	return $this->showError('404', 'error', 'b_options element without operator');
9465: 						$options_element[0] = trim($options_element[0]);
9466: 						switch ($options_element[0]){
9467: 							case '=':
9468: 								if (!isset($options_element[1])) return $this->showError('404', 'error', 'empty b_options element keys for assignment');
9469: 								if (!is_array($options_element[1])) return $this->showError('404', 'error', 'b_options element keys for assignment not array');
9470: 								if (!isset($options_element[2])) return $this->showError('404', 'error', 'empty b_options element value for assignment');
```

### `editTicket` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator, 6=Usher`
- source range: `19769-20141`
```php
19769: 		public function editTicket($id_trip = "", $data = "", $s_data = array(), $s_data_stt = array(), $s_data_sc = array())
19770: 		{
19771: 			if (empty($_SESSION[UID])) {
19772: 				return $this->showError('404', 'error', 'unauthorized access');
19773: 			}
19774: 
19775: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
19776: 
19777: 			if (empty($data)) 
19778: 			{
19779: 				return $this->showError('404', 'error', 'empty data');
19780: 			}
19781: 
19782: 			$data = json_decode($data,true);
19783: 			
19784: 			if (empty($data) || !is_array($data)) 
19785: 			{
19786: 				return $this->showError('404', 'error', 'wrong data');
19787: 			}
19788: 
19789: 			$s = "SELECT			
19790: 					`trip`.`id_trip`,
19791: 					`trip`.`driver`,
19792: 					`trip`.`from`
19793: 				FROM `trip`
19794: 				WHERE	
19795: 					`trip`.`id_trip` = '" . $id_trip . "'
19796: 				LIMIT 1
19797: 				";
19798: 			
19799: 			$q = query($s);
19800: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
19801: 
19802: 			$d = fetch_assoc($q);
19803: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
19804: 			$id_schedule = explode(chr(0),$d['from']);
19805: 			if (count($id_schedule) > 1 || $id_schedule[0] == 'sc_id')
19806: 			{
19807: 				$id_schedule = $id_schedule[1];
19808: 			}
19809: 			else
19810: 			{
19811: 				return $this->showError('404', 'error', 'sc_id not defined');
19812: 			}
19813: 			$is_usher = false;
19814: 			if ($this->id_role != 4 && ($d['driver'] != $_SESSION[UID] || $this->id_role == 6)) 
19815: 			{
19816: 				if ($this->id_role == 6)
19817: 				{
19818: 				
19819: 					$s = "SELECT
19820: 							`id_user` as u_id,
19821: 							`id_schedule` as sc_id
19822: 						FROM `users`
19823: 						WHERE
19824: 							`id_user` = '" . $_SESSION[UID]. "'
19825: 						";
19826: 
19827: 					$q = query($s);
19828: 					if ($q === false) return $this->showError('404', 'error', 'usher select failed');
19829: 
19830: 					$d_u = fetch_assoc($q);
19831: 					if (empty($d_u['sc_id']) || $d_u['sc_id'] != $id_schedule) return $this->showError('404', 'error', 'usher for other schedule');
19832: 					$is_usher = true;
19833: 				}
19834: 				else
19835: 				{
19836: 					return $this->showError('404', 'error', 'user is not author');
19837: 				}
19838: 			}
19839: 
19840: 			$out = array();
19841: 			$s_insert = array();
19842: 			$blocks = array();
19843: 			$delete_sql = array();
19844: 			$warning = array();
19845: 			$log_insert = array();
19846: 			foreach($data as $i=>$arr)
19847: 			{
19848: 				if (empty($arr['seat'])) {$warning["$i"] = 'empty seat'; continue;}
19849: 				$filtered_data = array();
19850: 				$filtered_data['id_seat'] = $arr['seat'];
19851: 				if ($is_usher === true)
19852: 				{
19853: 					if (isset($arr['pass']))
19854: 					{
19855: 						$arr = array('seat' => $arr['seat'], 'pass' => $arr['pass']);
19856: 					}
19857: 					else
19858: 					{
19859: 						$arr = array('seat' => $arr['seat']);
19860: 					}
19861: 				}
19862: 				elseif ($this->id_role != 4)
19863: 				{
19864: 					unset($arr['pass']);
19865: 				}
19866: 				if (!empty($arr['del']))
19867: 				{
19868: 					$delete_sql[] = $arr['seat'];
19869: 					continue;
19870: 				}
19871: 				if (isset($arr['price']))
19872: 				{
19873: 					if ($arr['price'] === "")
19874: 					{
19875: 						$filtered_data['currency'] = NULL;
19876: 						$filtered_data['tariff'] = NULL;
19877: 					}
19878: 					else
19879: 					{
19880: 						$tariff = explode(' ',$arr['price']);
19881: 						if (count($tariff) > 1)
19882: 						{
19883: 							$filtered_data['currency'] = $tariff[1];
19884: 						}
19885: 						else
19886: 						{
19887: 							$filtered_data['currency'] = NULL;
19888: 						}
19889: 						$filtered_data['tariff'] = $tariff[0];
19890: 					}
19891: 				}
19892: 				if (array_key_exists('number',$arr))
19893: 				{
19894: 					$filtered_data['number'] = $arr['number'];
19895: 				}
19896: 				if (array_key_exists('code',$arr))
19897: 				{
19898: 					$filtered_data['code'] = $arr['code'];
19899: 				}
19900: 				if (array_key_exists('code_qr_base64',$arr))
19901: 				{
19902: 					$filtered_data['code_qr_base64'] = $arr['code_qr_base64'];
19903: 				}
19904: 
19905: 				if (empty($arr['new']))
19906: 				{
19907: 					$var_block = false;
19908: 					$sql_status = "";
19909: 					if (isset($arr['status']))
19910: 					{
19911: 						$status = $arr['status'];
19912: 						if ($status == 1)
19913: 						{
19914: 							$var_block = true;
19915: 							$sql_status = ",
19916: 								`status` = IF(`id_order` IS NULL,
19917: 									IF(`status` in (2,3),@block:=1,@block:=0) IS NOT NULL AND '$status',
19918: 									@block:=0 = 0 AND `status`)";
19919: /*							$q = query("BEGIN");
19920: 							if ($q === false) return $this->showError('404', 'error', "$i begin query failed");*/
19921: 						}
19922: 						else
19923: 						{
19924: 							$sql_status = ",
19925: 								`status` = IF(`id_order` IS NULL,'$status',`status`)";
19926: 						}
19927: 					}
19928: 
19929: 					$sql_pass = '';
19930: 					if (isset($arr['pass']))
19931: 					{
19932: 						$pass_val = empty($arr['pass']) ? 0 : 1;
19933: 						if ($pass_val === 0)
19934: 						{
19935: 							$pass_datetime = 'out_datetime';
19936: 						}
19937: 						else
19938: 						{
19939: 							$pass_datetime = 'pass_datetime';
19940: 						}
19941: 						$int_timestamp = time();
19942: 						$now_datetime = (new DateTime("@$int_timestamp"))->setTimezone(new DateTimeZone(MYSQL_TIME_ZONE))->format('Y-m-d H:i:s');
19943: 						$sql_pass = ",
19944: 							`pass` = IF(@trip_seat:=`id_trip_seat`,'$pass_val','$pass_val'),
19945: 							`$pass_datetime` = '$now_datetime'";
19946: 						$var_block = true;
19947: 					}
19948: 
19949: 					if (isset($arr['name']))
19950: 					{
19951: 						$filtered_data['id_seat'] = $arr['name'];
19952: 					}
19953: 					else
19954: 					{
19955: 						if (count(($filtered_data)) == 1 && empty($sql_status) && empty($sql_pass)) {$warning["$i"] = 'empty data'; continue;}
19956: 					}
19957: 
19958: 					$s = array();
19959: 					foreach ($filtered_data as $key => $value)
19960: 					{
19961: 					
19962: 						$s[] = "`" . $key . "` = " 
19963: 								   . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
19964: 					}
19965: 
19966: 					$s = "UPDATE `ticket`
19967: 						SET 
19968: 							" . implode(",\n", $s) ."$sql_status$sql_pass
19969: 						WHERE
19970: 							`id_seat` = '" . $arr['seat'] . "' AND `id_trip` = '" . $id_trip . "'
19971: 						";
19972: 
19973: 					$q = query($s);
19974: 
19975: 					if ($q === false) return $this->showError('404', 'error', "database $i update failed");
19976: 
19977: 					if ($var_block === true)
19978: 					{
19979: 						$s = "SELECT @block,@trip_seat";
19980: 
19981: 						$q = query($s);
19982: 						if ($q === false) return $this->showError('404', 'error', 'variable select failed');
19983: 						$d_var = fetch_assoc($q);
19984: 						if (!empty($block = $d_var['@block']))
19985: 						{
19986: 							$seat_b_r_s = explode(';',$filtered_data['id_seat']);						
19987: 							if (count($seat_b_r_s) < 4) return $this->showError('404', 'error', 'wrong seat');
19988: 							if (isset($blocks[$seat_b_r_s[1]])) $blocks[$seat_b_r_s[1]]++; else $blocks[$seat_b_r_s[1]] = 1;
19989: 						}
```

### `editTrip` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `12880-13565`
```php
12880: 		public function editTrip($data = "", $id_trip = "")
12881: 		{
12882: 			if (empty($_SESSION[UID])) {
12883: 				return $this->showError('404', 'error', 'unauthorized access');
12884: 			}
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
12889: 			}
12890: 
12891: 			if (empty($data)) 
12892: 			{
12893: 				return $this->showError('404', 'error', 'empty data');
12894: 			}
12895: 
12896: 			$data = json_decode($data,true);
12897: 			
12898: 			if (empty($data) || !is_array($data)) 
12899: 			{
12900: 				return $this->showError('404', 'error', 'wrong data');
12901: 			}
12902: 
12903: 			$allowed_fields = array(
12904: 									't_start_address'			=>		'from',
12905: 									't_start_latitude'			=>		'from_lat',
12906: 									't_start_longitude'			=>		'from_lng',
12907: 									't_destination_address'		=>		'to',
12908: 									't_destination_latitude'	=>		'to_lat',
12909: 									't_destination_longitude'	=>		'to_lng',			
12910: 									't_start_datetime_interval'	=>		'start_plan_datetime_interval',		
12911: 									't_start_datetime'			=>		'start_plan_datetime',
12912: 									't_complete_datetime'		=>		'complete_plan_datetime',
12913: 									't_options'					=>		'json',
12914: 									't_start_real_datetime'		=>		'start_datetime',
12915: 									't_complete_real_datetime'	=>		'complete_datetime',	
12916: 									't_looking_for_clients'		=>		'looking_for_clients',
12917: 									't_canceled'				=>		'canceled',
12918: 									'price_time_function'		=>		'id_price_time_function',
12919: 									'currency'					=>		'currency',
12920: 									'currency_priority'			=>		'currency_priority',	
12921: 									'fee'						=>		'fee',
12922: 									'tariff'					=>		'tariff',
12923: 									'tariff_priority'			=>		'tariff_priority'								
12924: 			);
12925: 
12926: 			$q = query("BEGIN");
12927: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
12928: 
12929: 			$s = "SELECT			
12930: 					`trip`.`id_trip`,
12931: 					`trip`.`driver`,
12932: 					`trip`.`from`,
12933: 					`trip`.`from_lat`,
12934: 					`trip`.`from_lng`,
12935: 					`trip`.`to`,
12936: 					`trip`.`to_lat`,
12937: 					`trip`.`to_lng`,
12938: 					`trip`.`start_plan_datetime_interval`,
12939: 					`trip`.`start_plan_datetime`,
12940: 					`trip`.`complete_plan_datetime`,
12941: 					`trip`.`start_datetime`,
12942: 					`trip`.`complete_datetime`,
12943: 					`trip`.`json`,
12944: 					`trip`.`looking_for_clients`,
12945: 					`trip`.`canceled`,
12946: 					IF(`order_trip`.`id_order` IS NULL,NULL,
12947: 						GROUP_CONCAT(
12948: 							CONCAT_WS(0x00,
12949: 								`order_trip`.`id_order`,
12950: 								`order_trip`.`offer_order_datetime`,
12951: 								`order_trip`.`select_trip_datetime`,
12952: 								`order`.`id_order_status`,
12953: 								IFNULL(od.`id_order_driver_status`,0x02)
12954: 							)
12955: 						SEPARATOR 0x01) 
12956: 					) as orders
12957: 				FROM `trip`
12958: 				LEFT JOIN `order_trip` USING (`id_trip`)
12959: 				LEFT JOIN `order` ON `order`.`id_order` = `order_trip`.`id_order`
12960: 				LEFT JOIN (
12961: 					SELECT
12962: 						`id_order_driver_status`,
12963: 						`id_order`
12964: 					FROM
12965: 						`order_driver`
12966: 					WHERE
12967: 						`id_user` = '" . $_SESSION[UID] . "'
12968: 					LIMIT 1
12969: 				) od ON od.`id_order` = `order`.`id_order`
12970: 				WHERE	
12971: 					`id_trip` = '" . $id_trip . "'
12972: 				GROUP BY `trip`.`id_trip`
12973: 				LIMIT 1
12974: 				FOR UPDATE
12975: 				";
12976: 
12977: 			$q = query($s);
12978: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
12979: 
12980: 			$d = fetch_assoc($q);
12981: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
12982: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
12983: 			{
12984: 				return $this->showError('404', 'error', 'user is not author');
12985: 			}
12986: 			if ($d['complete_datetime'] != "0000-00-00 00:00:00") return $this->showError('404', 'error', 'completed trip');
12987: 			if (!empty($d['canceled'])) return $this->showError('404', 'error', 'canceled trip');
12988: 
12989: 			$d['json'] = json_decode($d['json'],true);
12990: 
12991: 			if (!empty($d['orders']))
12992: 			{
12993: 				$d['orders'] = explode(chr(1),$d['orders']);
12994: 				foreach ($d['orders'] as $key=>$value)
12995: 				{
12996: 					$d['orders'][$key] = array();
12997: 					list(
12998: 						$d['orders'][$key]['id_order'],
12999: 						$d['orders'][$key]['offer_order_datetime'],
13000: 						$d['orders'][$key]['select_trip_datetime'],
13001: 						$d['orders'][$key]['id_order_status'],
13002: 						$d['orders'][$key]['id_order_driver_status']
13003: 						)= explode(chr(0),$value);
13004: 
13005: 					if ($d['orders'][$key]['id_order_driver_status'] === chr(2))
13006: 					{
13007: 						$d['orders'][$key]['id_order_driver_status'] = NULL;
13008: 					}
13009: 				}
13010: 
13011: 				unset($allowed_fields['t_start_address']);
13012: 				unset($allowed_fields['t_start_latitude']);
13013: 				unset($allowed_fields['t_start_longitude']);
13014: 				unset($allowed_fields['t_destination_address']);
13015: 				unset($allowed_fields['t_destination_latitude']);
13016: 				unset($allowed_fields['t_destination_longitude']);
13017: 				unset($allowed_fields['t_start_datetime_interval']);
13018: 				unset($allowed_fields['t_start_datetime']);
13019: 				unset($allowed_fields['t_complete_datetime']);
13020: 			}
13021: 
13022: 			$forbidden_fields = array();
13023: 			$affected_fields = array();
13024: 			$affected_keys = array();
13025: 			$filtered_data = array();
13026: 
13027: 			foreach ($data as $key => $value)
13028: 			{
13029: 				if (!empty($allowed_fields[$key]))
13030: 				{
13031: 					if (is_string($value)) $data[$key] = trim($value);
13032: 					$affected_fields[] = $key;
13033: 					$affected_keys[$key] = true;
13034: 					if ($allowed_fields[$key] !== true) $filtered_data[$allowed_fields[$key]] = $data[$key];
13035: 				}
13036: 				else
13037: 				{
13038: 					$forbidden_fields[] = $key;
13039: 				}
13040: 			}
13041: 
13042: 			if (!empty($filtered_data['canceled']))
13043: 			{
13044: 				if (empty($affected_keys['t_start_real_datetime']))	
13045: 				{
13046: 					$filtered_data['complete_datetime'] = 'now';
13047: 					$filtered_data['canceled'] = 1;
13048: 				}
13049: 			}
13050: 			else
13051: 			{
13052: 				if (!empty($affected_keys['t_canceled'])) $filtered_data['canceled'] = 0;
13053: 			}
13054: 
13055: 			if (!empty($affected_keys['t_looking_for_clients'])) 
13056: 			{
13057: 				if (empty($filtered_data['looking_for_clients']))
13058: 				{
13059: 					$filtered_data['looking_for_clients'] = 0;
13060: 				}
13061: 				else
13062: 				{
13063: 					$filtered_data['looking_for_clients'] = 1;
13064: 				}			
13065: 			}
13066: 
13067: 			if (!empty($affected_keys['t_start_datetime']))
13068: 			{
13069: 				switch ($filtered_data['start_plan_datetime']) {
13070: 					case 'now':
13071: 						preg_match('/^(\+|-)([0-9]{2})\:([0-9]{2})$/',MYSQL_TIME_ZONE,$parsed_time_zone);
13072: 						$filtered_data['start_plan_datetime'] = gmdate("Y-m-d H:i:s",time() + ($parsed_time_zone[1] === '+' ? 1 : -1)*((int)$parsed_time_zone[2]*60 + (int)$parsed_time_zone[3])*60);
13073: 						break;
13074: 					default:
13075: 						preg_match('/^([0-9]{4}-[0-9]{1,2}-[0-9]{1,2})\s+([0-9]{1,2}\:[0-9]{1,2}\:[0-9]{1,2})\s*((?:\+|-)[0-9]{2}\:[0-9]{2})$/',$filtered_data['start_plan_datetime'],$parsed_datetime);
13076: 						if (empty($parsed_datetime))
13077: 						{
13078: 							return $this->showError('404', 'error', 'wrong t_start_datetime');
13079: 						}
13080: 						else
13081: 						{
13082: 							$filtered_data['start_plan_datetime'] = $parsed_datetime[1] . " " . $parsed_datetime[2];
13083: 							if ($filtered_data['start_plan_datetime'] == '0000-00-00 00:00:00')
13084: 							{
13085: 								return $this->showError('404', 'error', '0000-00-00 00:00:00 t_start_datetime');
13086: 							}
13087: 							else
13088: 							{
13089: 								preg_match('/^(\+|-)([0-9]{2})\:([0-9]{2})$/',MYSQL_TIME_ZONE,$parsed_time_zone);
13090: 								$datetime_start_plan = gmdate("Y-m-d H:i:s",strtotime($filtered_data['start_plan_datetime'] .  $parsed_datetime[3]) + ($parsed_time_zone[1] === '+' ? 1 : -1)*((int)$parsed_time_zone[2]*60 + (int)$parsed_time_zone[3])*60);
13091: 							}
13092: 						}
13093: 				}
13094: 			}
13095: 			if (!empty($affected_keys['t_complete_datetime']))
13096: 			{
13097: 				switch ($filtered_data['complete_plan_datetime']) {
13098: 					case 'now':
13099: 						preg_match('/^(\+|-)([0-9]{2})\:([0-9]{2})$/',MYSQL_TIME_ZONE,$parsed_time_zone);
13100: 						$filtered_data['complete_plan_datetime'] = gmdate("Y-m-d H:i:s",time() + ($parsed_time_zone[1] === '+' ? 1 : -1)*((int)$parsed_time_zone[2]*60 + (int)$parsed_time_zone[3])*60);
```

### `editUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `864-1830`
```php
864: 		public function editUser($data = "", $id_user = "", $roles = array(), $users_props = array(), $field_types = array())
865: 		{
866: 			if (empty($_SESSION[UID])) {
867: 				return $this->showError('404', 'error', 'unauthorized access');
868: 			}	
869: 			if (empty($data)) 
870: 			{
871: 				return $this->showError('404', 'error', 'empty data');
872: 			}
873: 
874: 			$data = json_decode($data,true);
875: 			
876: 			if (empty($data) || !is_array($data)) 
877: 			{
878: 				return $this->showError('404', 'error', 'wrong data');
879: 			}
880: 
881: 			$allowed_fields = array(
882: 									'u_role'				=>		'id_role',
883: 									'u_name'				=>		'name',
884: 									'u_family'				=>		'family',
885: 									'u_middle'				=>		'middle',
886: 									'u_phone'				=>		'phone',
887: 									'u_phone_checked'		=>		'phone_is_verified',
888: 									'u_email'				=>		'email',
889: 									'u_email_checked'		=>		'email_is_verified',
890: 									'u_photo'				=>		'photo_link',
891: 									'u_lang'				=>		'id_lang',
892: 									'u_currency'			=>		'currency',
893: 									'u_city'				=>		'id_city',
894: 									'u_tips'				=>		'tips',
895: 									'u_lang_skills'			=>		'language_skills',
896: 									'u_description'			=>		'description',
897: 									'u_gps_software'		=>		'id_navigation',
898: 									'u_check_state'			=>		'id_verification_status',
899: 									'u_active'				=>		'active',
900: 									'u_is_deleted'			=>		'deleted',
901: 									'u_birthday'			=>		'birthday_date',
902: 									'out_drive'				=>		'out_order',
903: 									'out_address'			=>		'out_order_to',
904: 									'out_latitude'			=>		'out_order_to_lat',
905: 									'out_longitude'			=>		'out_order_to_lng',
906: 									'out_est_datetime'		=>		'out_order_complete_datetime',
907: 									'out_s_address'			=>		'out_order_from',
908: 									'out_s_latitude'		=>		'out_order_from_lat',
909: 									'out_s_longitude'		=>		'out_order_from_lng',
910: 									'out_passengers'		=>		'out_order_passengers_count',
911: 									'out_luggage'			=>		'out_order_luggage_count',
912: 									'ref_code'				=>		'referral_code',
913: 									'referrer_u_id'			=>		'referrer',
914: 									'u_tg'					=>		'tg',
915: 									'u_tg_checked'			=>		'tg_is_verified',
916: 									'u_upper'				=>		'id_user_upper'	,
917: 									'b_comments'			=> 		array(
918: 																		'table'		=>	'users_order_comment',
919: 																		'field'		=>	'id_order_comment'
920: 																	),
921: 									'b_services'			=> 		array(
922: 																		'table'		=>	'users_service',
923: 																		'field'		=>	'id_service'
924: 																	),
925: 									'b_location_classes'	=> 		array(
926: 																		'table'		=>	'users_order_location',
927: 																		'field'		=>	'id_order_location',
928: 																		'field_add'	=>	array(
929: 																						'name'		=>	'basic',
930: 																						'default'	=>	'1'
931: 																						)
932: 																	),
933: 									'u_details'				=>		'json',
934: 									'sc_id'					=>		'id_schedule',
935: 									'u_wa'					=>		'wa',
936: 									'u_wa_checked'			=>		'wa_is_verified'
937: 			);
938: 			if (empty($id_user) || $_SESSION[UID] == $id_user)
939: 			{	
940: 				$id_user = $_SESSION[UID];
941: 				$id_role = $_SESSION['id_role'];
942: 				$phone = $_SESSION['phone'];
943: 				$email = $_SESSION['email'];
944: 				$tg = $_SESSION['tg'];
945: 				$wa = $_SESSION['wa'];
946: 				switch ($this->id_role) {
947: 					case '1':
948: 						$allowed_fields = array(
949: 												'u_role'				=>		'id_role',
950: 												'u_name'				=>		'name',
951: 												'u_family'				=>		'family',
952: 												'u_middle'				=>		'middle',
953: 												'u_phone'				=>		'phone',
954: 												'u_email'				=>		'email',
955: 												'u_photo'				=>		'photo_link',
956: 												'u_lang'				=>		'id_lang',
957: 												'u_currency'			=>		'currency',
958: 												'ref_code'				=>		'referral_code',
959: 												'u_details'				=>		'json'
960: 						);
961: 						break;
962: 					case '2':
963: 						if ($_SESSION['id_verification_status'] == 3)
964: 						{
965: 							return $this->showError('404', 'error', 'driver with rejected verification');
966: 						}
967: 						if ($_SESSION['id_verification_status'] == 4)
968: 						{
969: 							return $this->showError('404', 'error', 'blocked driver');
970: 						}
971: 						if ($_SESSION['id_verification_status'] == 2)
972: 						{
973: 							$allowed_fields = array(
974: 										'u_role'				=>		'id_role',
975: 										'u_lang'				=>		'id_lang',
976: 										'u_currency'			=>		'currency',
977: 										'u_gps_software'		=>		'id_navigation',
978: 										'u_active'				=>		'active',
979: 										'out_drive'				=>		'out_order',
980: 										'out_address'			=>		'out_order_to',
981: 										'out_latitude'			=>		'out_order_to_lat',
982: 										'out_longitude'			=>		'out_order_to_lng',
983: 										'out_est_datetime'		=>		'out_order_complete_datetime',
984: 										'out_s_address'			=>		'out_order_from',
985: 										'out_s_latitude'		=>		'out_order_from_lat',
986: 										'out_s_longitude'		=>		'out_order_from_lng',
987: 										'out_passengers'		=>		'out_order_passengers_count',
988: 										'out_luggage'			=>		'out_order_luggage_count',
989: 										'ref_code'				=>		'referral_code',
990: 										'b_comments'			=> 		array(
991: 																			'table'		=>	'users_order_comment',
992: 																			'field'		=>	'id_order_comment'
993: 																		),
994: 										'b_services'			=> 		array(
995: 																			'table'		=>	'users_service',
996: 																			'field'		=>	'id_service'
997: 																		),
998: 										'b_location_classes'	=> 		array(
999: 																			'table'		=>	'users_order_location',
1000: 																			'field'		=>	'id_order_location',
1001: 																			'field_add'	=>	array(
1002: 																							'name'		=>	'basic',
1003: 																							'default'	=>	'1'
1004: 																							)
1005: 																		),
1006: 										'u_details'				=>		'json'
1007: 							);
1008: 						}
1009: 						elseif (empty($_SESSION['id_verification_status']) 
1010: 								|| $_SESSION['id_verification_status'] == 1)
1011: 						{
1012: 							$allowed_fields = array(
1013: 													'u_role'				=>		'id_role',
1014: 													'u_name'				=>		'name',
1015: 													'u_family'				=>		'family',
1016: 													'u_middle'				=>		'middle',
1017: 													'u_phone'				=>		'phone',
1018: 													'u_email'				=>		'email',
1019: 													'u_photo'				=>		'photo_link',
1020: 													'u_city'				=>		'id_city',
1021: 													'u_lang_skills'			=>		'language_skills',
1022: 													'u_description'			=>		'description',
1023: 													'u_birthday'			=>		'birthday_datetime',
1024: 													'ref_code'				=>		'referral_code',
1025: 													'u_details'				=>		'json'
1026: 							);
1027: 						}			
1028: 						break;
1029: 					case '4':
1030: 						break;
1031: 					case '5':
1032: 						$allowed_fields = array(
1033: 												'u_role'				=>		'id_role',
1034: 												'u_name'				=>		'name',
1035: 												'u_family'				=>		'family',
1036: 												'u_middle'				=>		'middle',
1037: 												'u_phone'				=>		'phone',
1038: 												'u_email'				=>		'email',
1039: 												'u_photo'				=>		'photo_link',
1040: 												'u_lang'				=>		'id_lang',
1041: 												'u_currency'			=>		'currency',
1042: 												'ref_code'				=>		'referral_code',
1043: 												'u_details'				=>		'json'
1044: 						);
1045: 						break;
1046: 					case '6':
1047: 						$allowed_fields = array(
1048: 						);
1049: 						break;
1050: 					default:
1051: 						return $this->showError('404', 'error', 'wrong user role');
1052: 				}
1053: 			}
1054: 			else
1055: 			{
1056: 				if ($this->id_role != 4) 
1057: 				{
1058: 					return $this->showError('404', 'error', 'not enough rights');
1059: 				}
1060: 				
1061: 				$s = "SELECT 
1062: 						`id_role`,
1063: 						`id_user`,
1064: 						`phone`,
1065: 						`email`,
1066: 						`id_verification_status`,
1067: 						`tg`,
1068: 						`wa`
1069: 					FROM `users`
1070: 					WHERE 
1071: 						`id_user` = '" . $id_user . "'
1072: 					LIMIT 1
1073: 					";
1074: 
1075: 				$q = query($s);
1076: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
1077: 				$d = fetch_assoc($q);
1078: 				
1079: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');
1080: 
1081: 				$id_role = $d['id_role'];			
1082: 				$phone = $d['phone'];
1083: 				$email = $d['email'];
1084: 				$tg = $d['tg'];
```

### `editWaitingTime` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 5=Agent`
- source range: `8734-8829`
```php
8734: 		public function editWaitingTime($id_order = "", $previous = NULL, $additional = NULL)
8735: 		{
8736: 			if ($previous === NULL) return $this->showError('404', 'error', 'empty previous');
8737: 			if ($additional === NULL) $additional = $this->constant['waiting_interval_add'];
8738: 
8739: 			$additional = (int)$additional;
8740: 			if (empty($additional)) return $this->showError('404', 'error', 'empty additional');
8741: 			if (empty($_SESSION[UID])) {
8742: 				return $this->showError('404', 'error', 'unauthorized access');
8743: 			}
8744: 			if ($this->id_role != 1 && $this->id_role != 5)
8745: 			{
8746: 				return $this->showError('404', 'error', 'wrong user role');
8747: 			}
8748: 
8749: 			$q = query("BEGIN");
8750: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
8751: 
8752: 			$s = "SELECT
8753: 					`order`.`id_order`,
8754: 					`order`.`client`,
8755: 					`order`.`id_order_status`,
8756: 					TIMESTAMPDIFF(SECOND,IF(`order`.`datetime_start_plan` = 0,`order`.`create_datetime`,`order`.`datetime_start_plan`),`order`.`max_waiting_datetime`) as max_waiting_time,
8757: 					MAX(`order_waiting_time`.`index`) as max_index
8758: 				FROM `order` 
8759: 				LEFT JOIN `order_waiting_time` USING (`id_order`)		
8760: 				WHERE	
8761: 					`order`.`id_order` = '" . $id_order . "'
8762: 				LIMIT 1
8763: 				FOR UPDATE
8764: 				";
8765: 
8766: 			$q = query($s);
8767: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8768: 
8769: 			$d = fetch_assoc($q);
8770: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8771: 			if ($d['client'] != $_SESSION[UID]) 
8772: 			{
8773: 				return $this->showError('404', 'error', 'user is not author');
8774: 			}
8775: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 5 && $d['id_order_status'] != 6) 
8776: 			{
8777: 				return $this->showError('404', 'error', 'wrong booking state');
8778: 			}	
8779: 
8780: 			$d['max_waiting_time'] = (int)$d['max_waiting_time'];
8781: 			$previous = (int)$previous;
8782: 			if ($d['max_waiting_time'] !== $previous)
8783: 			{
8784: 				return $this->showError('404', 'error', 'wrong previous');
8785: 			}
8786: 			$d['max_index'] = (int)$d['max_index'];
8787: 
8788: 			$s = "INSERT INTO `order_waiting_time`
8789: 				SET 
8790: 					`id_order` = '" . $id_order . "',
8791: 					`interval` = '" . $additional . "',
8792: 					`datetime` = now(),
8793: 					`index` = '" . ($d['max_index'] + 1) . "'
8794: 				";
8795: 
8796: 			$q = query($s);
8797: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');
8798: 			
8799: 			$s = "UPDATE `order`
8800: 				SET 
8801: 					`max_waiting_datetime` =  IF(`datetime_start_plan` = 0,`create_datetime`,`datetime_start_plan`) + INTERVAL '" . ($previous + $additional) . "' SECOND,
8802: 					`last_edit_datetime` = now(),
8803: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
8804: 				WHERE
8805: 					`id_order` = '" . $id_order . "'
8806: 				";
8807: 
8808: 			query($s);
8809: 	
8810: 			global $link;
8811: 
8812: 			if (mysqli_affected_rows($link) === -1) 
8813: 			{
8814: 				return $this->showError('404', 'error', 'database update failed');
8815: 			}
8816: 			elseif (mysqli_affected_rows($link) === 0) 
8817: 			{
8818: 				return $this->showError('404', 'error', 'modified data not found');
8819: 
8820: 			}
8821: 
8822: 			$q = query("COMMIT");
8823: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
8824: 
8825: 			return array(
8826: 				'code' 		=>	'200',
8827: 				'status' 	=>	'success'
8828: 			);
8829: 		}
```

### `getDropboxFileData` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `23433-23582`
```php
23433: 		public function getDropboxFileData($id_dropbox_link = '', $private = '', $deleted = '')
23434: 		{
23435: 			if ($id_dropbox_link == 'null') $id_dropbox_link = NULL;
23436: 			if (empty($_SESSION[UID]))
23437: 			{
23438: 				$sql_where = '1 = 1';
23439: 				if (!empty($id_dropbox_link))
23440: 				{
23441: 					$sql_where .= " AND `id_dropbox_link` in ($id_dropbox_link)";
23442: 				}
23443: 				$s = "SELECT
23444: 						`id_dropbox_link` as dl_id,
23445: 						`json`
23446: 					FROM `dropbox_link` 				
23447: 					WHERE	
23448: 						$sql_where AND `private` = '-1' AND `deleted` = 0
23449: 					ORDER BY `id_dropbox_link`
23450: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
23451: 					";
23452: 
23453: 				$q = query($s);
23454: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
23455: 
23456: 				$out = array('dropbox files' => array());
23457: 				while ($d = fetch_assoc($q))
23458: 				{
23459: 					$d['json'] = json_decode($d['json'],true);
23460: 					$d['json'] = array(
23461: 								'name' 			=> $d['json']['name'], 
23462: 								'name_upload' 	=> $d['json']['name_upload'], 
23463: 								'type' 			=> $d['json']['type'], 
23464: 								'size' 			=> $d['json']['size']
23465: 					);
23466: /*					unset($d['json']['response']);
23467: 					unset($d['json']['response_del']);
23468: 					unset($d['json']['owner']);*/
23469: 					$out['dropbox files'][$d['dl_id']] = $d;
23470: 				}
23471: 				if (empty($this->associativeArray)) $out['contacts'] =  array_values($out['contacts']);
23472: 			}
23473: 			elseif ($this->id_role == 4)
23474: 			{
23475: 				$sql_where = '1 = 1';
23476: 				if (!empty($id_dropbox_link))
23477: 				{
23478: 					$sql_where .= " AND `id_dropbox_link` in ($id_dropbox_link)";
23479: 				}
23480: 				if ($private != '')
23481: 				{
23482: 					$sql_where .= " AND `private` in ($private)";
23483: 				}
23484: 				if ($deleted != '')
23485: 				{
23486: 					$sql_where .= " AND `deleted` in ($deleted)";
23487: 				}
23488: 				$s = "SELECT
23489: 						`dropbox_link`.`id_dropbox_link` dl_id,
23490: 						`dropbox_link`.`json`,
23491: 						`dropbox_link`.`private`,
23492: 						`dropbox_link`.`deleted`,
23493: 						GROUP_CONCAT(`users_dropbox_link`.`id_user` SEPARATOR ',') as users,
23494: 						GROUP_CONCAT(IF(`users_dropbox_link`.`owner` = 1,`users_dropbox_link`.`id_user`,NULL) SEPARATOR ',') as u_id
23495: 					FROM `dropbox_link`
23496: 					LEFT JOIN `users_dropbox_link` USING (`id_dropbox_link`)
23497: 					WHERE	
23498: 						$sql_where
23499: 					GROUP BY
23500: 						`dropbox_link`.`id_dropbox_link`
23501: 					ORDER BY `id_dropbox_link`
23502: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
23503: 					";
23504: 	
23505: 				$q = query($s);
23506: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
23507: 
23508: 				$out = array('dropbox files' => array());
23509: 				while ($d = fetch_assoc($q))
23510: 				{
23511: 					$d['json'] = json_decode($d['json'],true);
23512: 					if (!empty($d['users']))
23513: 					{
23514: 						$d['users'] = explode(',',$d['users']);
23515: 					}
23516: 					$out['dropbox files'][$d['dl_id']] = $d;
23517: 				}
23518: 				if (empty($this->associativeArray)) $out['contacts'] =  array_values($out['contacts']);
23519: 			}
23520: 			else
23521: 			{
23522: 				$sql_where = '1 = 1';
23523: 				if (!empty($id_dropbox_link))
23524: 				{
23525: 					$sql_where .= " AND `dropbox_link`.`id_dropbox_link` in ($id_dropbox_link)";
23526: 				}
23527: 				if ($private != '')
23528: 				{
23529: 					$sql_where .= " AND `dropbox_link`.`private` in ($private)";
23530: 				}
23531: 
23532: 				$s = "SELECT
23533: 						`dropbox_link`.`id_dropbox_link` as dl_id,
23534: 						`dropbox_link`.`json`,
23535: 						`dropbox_link`.`private`,
23536: 						IF(udl.`id_user` = '" . $_SESSION[UID] . "',(SELECT
23537: 							GROUP_CONCAT(`users_dropbox_link`.`id_user` SEPARATOR ',')
23538: 						 FROM `users_dropbox_link`
23539: 						 WHERE `users_dropbox_link`.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link`
23540: 						),'') as users,
23541: 						udl.`id_user` as u_id
23542: 					FROM `dropbox_link`
23543: 					LEFT JOIN `users_dropbox_link` udl ON udl.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link` AND udl.`owner` = 1
23544: 					WHERE	
23545: 						$sql_where AND (udl.`id_user` = '" . $_SESSION[UID] . "' OR `dropbox_link`.`private` in (-1,0)) AND `deleted` = 0
23546: 					GROUP BY
23547: 						`dropbox_link`.`id_dropbox_link`
23548: 					ORDER BY `dropbox_link`.`id_dropbox_link`
23549: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
23550: 					";
23551: 
23552: 				$q = query($s);
23553: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
23554: 
23555: 				$out = array('dropbox files' => array());
23556: 				while ($d = fetch_assoc($q))
23557: 				{
23558: 					$d['json'] = json_decode($d['json'],true);
23559: 					if ($d['u_id'] != $_SESSION[UID])
23560: 					{
23561: 						$d['json'] = array(
23562: 									'name' 			=> $d['json']['name'], 
23563: 									'name_upload' 	=> $d['json']['name_upload'], 
23564: 									'type' 			=> $d['json']['type'], 
23565: 									'size' 			=> $d['json']['size']
23566: 						);
23567: 					}
23568: 					if (!empty($d['users']))
23569: 					{
23570: 						$d['users'] = explode(',',$d['users']);
23571: 					}
23572: 					$out['dropbox files'][$d['dl_id']] = $d;
23573: 				}
23574: 				if (empty($this->associativeArray)) $out['contacts'] =  array_values($out['contacts']);
23575: 			}
23576: 			
23577: 			return array(
23578: 				'code' 		=>	'200',
23579: 				'status' 	=>	'success',		
23580: 				'data' 		=>	$out
23581: 			);
23582: 		}
```

### `getTaskLog` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `23178-23231`
```php
23178: 		public function getTaskLog($id_task_list = '')
23179: 		{
23180: 			if (empty($_SESSION[UID])) {
23181: 				return $this->showError('404', 'error', 'unauthorized access');
23182: 			}
23183: 
23184: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23185: 
23186: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
23187: 
23188: 			$s = "SELECT 
23189: 				`id_task_list` as tl_id,
23190: 				`id_account` as account,
23191: 				`id_task_comment` as task_comment,
23192: 				`id_task_action_function` as task_action_function,
23193: 				`id_task_action` as task_action,
23194: 				`id_task_action_control` as task_action_control,
23195: 				`comment` as custom_comment,
23196: 				`account` as custom_account,
23197: 				`send_account`,
23198: 				`json`,
23199: 				`last_edit_timestamp` as timestamp_edited,
23200: 				`create_timestamp` as timestamp_created,
23201: 				UNIX_TIMESTAMP(`last_edit_timestamp`) as unix_timestamp_edited,
23202: 				UNIX_TIMESTAMP(`create_timestamp`) as unix_timestamp_created,
23203: 				`last_edit_datetime` as edited,
23204: 				`create_datetime` as created
23205: 			FROM `task_list_log`
23206: 			WHERE
23207: 				`id_task_list` = '$id_task_list'
23208: 			ORDER BY `create_datetime` 
23209: 			";
23210: 			$q = query($s);
23211: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
23212: 
23213: 			$out = array('log' => array());
23214: 			while ($d = fetch_assoc($q))
23215: 			{
23216: 				if(!empty($d['custom_account'])) $d['custom_account'] = json_decode($d['custom_account'],true);
23217: 				if(!empty($d['send_account'])) $d['send_account'] = json_decode($d['send_account'],true);
23218: 				$d['json'] = json_decode($d['json'],true);
23219: 				add_time_zone($d['timestamp_edited']);
23220: 				add_time_zone($d['timestamp_created']);
23221: 				add_time_zone($d['edited']);
23222: 				add_time_zone($d['created']);
23223: 				$out['log'][] = $d;
23224: 			}
23225: 
23226: 			return array(
23227: 				'code' 		=>	'200',
23228: 				'status' 	=>	'success',		
23229: 				'data' 		=>	$out
23230: 			);
23231: 		}
```

### `getTicketData` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 4=Administrator`
- source range: `23584-24175`
```php
23584: 		public function getTicketData($id_schedule = "", $id_trip = '', $seat = '', $taken = false, $code_qr = false, $scheme = false, $full = false, $langs = array(), $price_time_functions = array())
23585: 		{
23586: 			if (empty($id_schedule)) return $this->showError('404', 'error', 'empty sc_id');
23587: 			$sql_where = "`ticket`.`id_schedule` = '$id_schedule'";
23588: 			$sql_field = $sql_field_code_qr = $sql_left_join = '';
23589: 			if ($code_qr == true){
23590: 				$sql_field_code_qr = "`ticket`.`code_qr_base64`,";
23591: 			}
23592: 
23593: 			if (!empty($id_trip)) 
23594: 			{
23595: 				$sql_where .= "`ticket`.`id_trip`in ($id_trip)";
23596: 			}
23597: 
23598: 			if (!empty($seat)) 
23599: 			{
23600: 				$seat = json_decode($seat,true);
23601: 				
23602: 				if (empty($seat) || !is_array($seat)) 
23603: 				{
23604: 					return $this->showError('404', 'error', 'wrong seat');
23605: 				}
23606: 
23607: 				$sql_seat = array();
23608: 				foreach($seat as $s_el)
23609: 				{
23610: 					$sql_seat[] = "'$s_el'";
23611: 				}
23612: 				$sql_seat = implode(',',$sql_seat);
23613: 				$sql_where .= " AND `ticket`.`id_seat` in ($sql_seat)";
23614: 			}
23615: 
23616: 			$sql_where_add = " AND (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NULL AND `ticket`.`status` not in (2,3))";
23617: 			if ($taken == true && ($this->id_role == 1 || $this->id_role == 2 || $this->id_role == 4))
23618: 			{
23619: 				if ($this->id_role == 4)
23620: 				{
23621: 					$sql_field = $sql_field_code_qr;
23622: 					$sql_field .= "
23623: 					`ticket`.`code`,
23624: 					`ticket`.`pass`,
23625: 					`ticket`.`pass_datetime`,
23626: 					`ticket`.`out_datetime`,";
23627: 					$sql_where_add = " AND (`ticket`.`id_order` IS NOT NULL OR `cart`.`id_user` IS NOT NULL OR `ticket`.`status` in (2,3))";
23628: 				}
23629: 				elseif ($this->id_role == 1)
23630: 				{
23631: 					if ($code_qr == true){
23632: 						$sql_field = "IF(`ticket`.`id_order` IS NOT NULL,`ticket`.`code_qr_base64`,NULL) as 'code_qr_base64',";
23633: 					}
23634: 					$sql_field .= "
23635: 					IF(`ticket`.`id_order` IS NOT NULL,`ticket`.`code`,NULL) as 'code',";
23636: 					$sql_where_add = " AND ((`ticket`.`id_order` IS NOT NULL AND `order`.`client` = '{$_SESSION[UID]}') OR (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NOT NULL AND `cart`.`id_user` = '{$_SESSION[UID]}'))";
23637: 					$sql_left_join = "LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`";
23638: 				}
23639: 				elseif ($this->id_role == 2)
23640: 				{
23641: 					$sql_field = $sql_field_code_qr;
23642: 					$sql_field .= "
23643: 					`ticket`.`code`,";
23644: 					$sql_where_add = " AND `trip`.`driver` = '{$_SESSION[UID]}' AND (`ticket`.`id_order` IS NOT NULL OR `cart`.`id_user` IS NOT NULL OR `ticket`.`status` in (2,3))";
23645: 					$sql_left_join = "LEFT JOIN `trip` ON `trip`.`id_trip` = `ticket`.`id_trip`";
23646: 				}
23647: 			}
23648: 			else
23649: 			{
23650: 				if ($this->id_role == 4)
23651: 				{
23652: 					$sql_field = $sql_field_code_qr;
23653: 					$sql_field .= "
23654: 					`ticket`.`code`,
23655: 					`ticket`.`pass`,
23656: 					`ticket`.`pass_datetime`,
23657: 					`ticket`.`out_datetime`,";
23658: 					$sql_where_add = "";
23659: 				}
23660: 				elseif ($this->id_role == 1)
23661: 				{
23662: 					if ($code_qr == true){
23663: 						$sql_field = "IF(`ticket`.`id_order` IS NOT NULL,`ticket`.`code_qr_base64`,NULL) as 'code_qr_base64',";
23664: 					}
23665: 					$sql_field .= "
23666: 					IF(`ticket`.`id_order` IS NOT NULL,`ticket`.`code`,NULL) as 'code',";
23667: 					$sql_where_add = " AND ((`ticket`.`id_order` IS NOT NULL AND `order`.`client` = '{$_SESSION[UID]}') OR (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NOT NULL AND `cart`.`id_user` = '{$_SESSION[UID]}') OR (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NULL AND `ticket`.`status` not in (2,3)))";
23668: 					$sql_left_join = "LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`";
23669: 				}
23670: 				elseif ($this->id_role == 2)
23671: 				{
23672: 					$sql_field = $sql_field_code_qr;
23673: 					$sql_field .= "
23674: 					`ticket`.`code`,";
23675: 					$sql_where_add = " AND `trip`.`driver` = '{$_SESSION[UID]}'";
23676: 					$sql_left_join = "LEFT JOIN `trip` ON `trip`.`id_trip` = `ticket`.`id_trip`";
23677: 				}
23678: 			}
23679: 			$sql_where .= $sql_where_add;
23680: 	
23681: 			$s = "SELECT
23682: 					`ticket`.`id_schedule` as sc_id,
23683: 					`ticket`.`id_seat` as seat,
23684: 					`ticket`.`id_trip` as t_id,
23685: 					`ticket`.`id_order` as b_id,
23686: 					`ticket`.`tariff`,
23687: 					`ticket`.`currency`,
23688: 					`ticket`.`id_trip_seat` as ti_t_id,
23689: 					`ticket`.`number`,
23690: 					`ticket`.`status`,
23691: 					$sql_field
23692: 					`cart`.`id_user` as cart_u_id,
23693: 					`cart`.`booking_limit` as cart_limit
23694: 				FROM `ticket`
23695: 				LEFT JOIN `cart` ON `cart`.`product` = `ticket`.`id_trip` AND `cart`.`property` = `ticket`.`id_trip_seat` AND `cart`.`booking_limit` > now()
23696: 				$sql_left_join
23697: 				WHERE 
23698: 					 $sql_where
23699: 				";
23700: 
23701: 			$q = query($s);
23702: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
23703: 			
23704: 			$out = array(
23705: 				'ticket' => array(),
23706: 				'schedule' => array(),
23707: 				'trip' => array(),
23708: 				'booking' => array(),
23709: 				'user' => array(),
23710: 				'price' => array(),
23711: 				'stadiums' => array(),
23712: 				'teams' => array(),
23713: 				'tournaments' => array(),
23714: 				'cities' => array(),
23715: 				'countries' => array(),
23716: 				'regions' => array()
23717: 			);
23718: 			$schedule = $trip = $booking = $user = $stadiums = $teams = $tournaments = $cities = $countries = $regions = array();
23719: 			while ($d = fetch_assoc($q))
23720: 			{
23721: 				$schedule[$d['sc_id']] = true;
23722: 				$trip[$d['t_id']] = true;
23723: 				if (!empty($d['b_id'])) $booking[$d['b_id']] = true;
23724: 				if (!empty($d['cart_u_id'])) $user[$d['cart_u_id']] = true;
23725: 				add_time_zone($d['cart_limit']);
23726: 				if (!empty($d['pass_datetime'])) add_time_zone($d['pass_datetime']);
23727: 				if (!empty($d['out_datetime'])) add_time_zone($d['out_datetime']);
23728: 				$out['ticket'][] = $d;
23729: 			}
23730: 
23731: 			$sql_name = $sql_description = $sql_address = $sql_country = array();
23732: 			foreach ($langs as $lang)
23733: 			{
23734: 				$sql_name[] = "`name_{$lang['iso']}` as {$lang['iso']}";
23735: 				$sql_description[] = "`description_{$lang['iso']}` as about_{$lang['iso']}";
23736: 				$sql_address[] = "`address_{$lang['iso']}` as address_{$lang['iso']}";
23737: 				$sql_country[] = "`country_name_{$lang['iso']}` as {$lang['iso']}";
23738: 			}
23739: 			$sql_name = implode(',',$sql_name);
23740: 			$sql_description = implode(',',$sql_description);
23741: 			$sql_address = implode(',',$sql_address);
23742: 			$sql_country = implode(',',$sql_country);
23743: 
23744: 			if (!empty($schedule))
23745: 			{
23746: 				$s = "SELECT
23747: 						`id_schedule` as sc_id,
23748: 						`team1`,
23749: 						`team2`,
23750: 						`id_stadium` as stadium,
23751: 						`id_tournament` as tournament,
23752: 						`start_datetime` as datetime,
23753: 						`duration`,
23754: 						`only_date`,
23755: 						`top`,
23756: 						`options`,
23757: 						`time_zone`,
23758: 						`id_price_time_function` as price_time_function,
23759: 						`currency`,
23760: 						`currency_priority`,
23761: 						`fee`,
23762: 						`tariff`,
23763: 						`tariff_priority`,
23764: 						`code_ean_base64`
23765: 					FROM `schedule`
23766: 					WHERE
23767: 						`id_schedule` in (" . implode(',',array_keys($schedule)) . ")
23768: 					";
23769: 
23770: 				$q = query($s);
23771: 
23772: 				if ($q === false) return $this->showError('404', 'error', 'schedule select failed');
23773: 				while ($d = fetch_assoc($q))
23774: 				{
23775: 					add_time_zone($d['datetime']);
23776: 					$d['options'] = json_decode($d['options'],true);
23777: 					if (!empty($d['team1'])) $teams[$d['team1']] = true;
23778: 					if (!empty($d['team2'])) $teams[$d['team2']] = true;
23779: 					if (!empty($d['stadium'])) $stadiums[$d['stadium']] = true;
23780: 					if (!empty($d['tournament'])) $tournaments[$d['tournament']] = true;
23781: 					$out['schedule'][$d['sc_id']] = $d;
23782: 				}
23783: 			}
23784: 
23785: 			if (!empty($trip))
23786: 			{
23787: 				$s = "SELECT
23788: 						`id_trip` as t_id,
23789: 						`driver` as u_id,
23790: 						`id_schedule` as sc_id,
23791: 						`create_datetime` as t_create_datetime,
23792: 						`id_price_time_function` as price_time_function,
23793: 						`currency`,
23794: 						`currency_priority`,
23795: 						`fee`,
23796: 						`tariff`,
23797: 						`tariff_priority`
23798: 					FROM `trip`
23799: 					WHERE
23800: 						`id_trip` in (" . implode(',',array_keys($trip)) . ")
23801: 					";
23802: 
23803: 				$q = query($s);
23804: 
```

### `includeTemplate` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20255-20299`
```php
20255: 		public function includeTemplate($template = "", $is_var = false, $script_templates = array())
20256: 		{	
20257: 			if (!isset($_POST['s_t_data'])) $_POST['s_t_data'] = array();
20258: 
20259: 			if (empty($_SESSION[UID])) {
20260: 				return $this->showError('404', 'error', 'unauthorized access');
20261: 			}			
20262: 
20263: 			if (empty($template)) return $this->showError('404', 'error', 'empty template');
20264: 
20265: 			if (empty($is_var))
20266: 			{
20267: 				if (!isset($script_templates[$template])) return $this->showError('404', 'error', 'template not found');
20268: 				$script_file = $_SERVER['DOCUMENT_ROOT'] . $script_templates[$template]['file'];
20269: 				$script_template =  $script_templates[$template];
20270: 			}
20271: 			else
20272: 			{
20273: 				foreach($script_templates as $script_template)
20274: 				{
20275: 					if ($script_template['var'] == $template) 
20276: 					{
20277: 						$script_file = $_SERVER['DOCUMENT_ROOT'] . $script_template['file'];
20278: 						break;
20279: 					}
20280: 				}
20281: 				if (empty($script_file)) return $this->showError('404', 'error', 'script template not found');
20282: 			}
20283: 
20284: 			if (!empty($script_template['only_admin']) && $this->id_role != 4)
20285: 			{
20286: 				return $this->showError('404', 'error', 'not enough rights');
20287: 			}
20288: 
20289: 			$out = array();
20290: 
20291: 			if (!is_file($script_file))	return $this->showError('404', 'error', 'include failed');
20292: 			include($script_file);
20293: 
20294: 			return array(
20295: 				'code' 		=>	'200',
20296: 				'status' 	=>	'success',
20297: 				'data'		=>	$out
20298: 			);
20299: 		}
```

### `offerOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 5=Agent`
- source range: `13567-13689`
```php
13567: 		public function offerOrder($id_order = "", $id_user = "", $trips = "")
13568: 		{
13569: 			if (empty($_SESSION[UID])) {
13570: 				return $this->showError('404', 'error', 'unauthorized access');
13571: 			}
13572: 			if ($this->id_role != 1 && $this->id_role != 5)
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
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
13593: 			$q = query($s);
13594: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
13595: 
13596: 			$d = fetch_assoc($q);
13597: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
13598: 			if ($d['id_order_status'] == 3 || $d['id_order_status'] == 4) return $this->showError('404', 'error', 'wrong booking state');
13599: 			if ($d['client'] != $_SESSION[UID]) 
13600: 			{
13601: 				return $this->showError('404', 'error', 'user is not author');
13602: 			}
13603: 			if (!empty($d['u_id'])) 
13604: 			{
13605: 				return $this->showError('404', 'error', $id_user . ' is performer');
13606: 			}
13607: 
13608: 			$s = "SELECT
13609: 					`id_user`
13610: 				FROM `users` 		
13611: 				WHERE	
13612: 					`id_user` = '" . $id_user. "'
13613: 				LIMIT 1
13614: 				";
13615: 
13616: 			$q = query($s);
13617: 			if ($q === false) return $this->showError('404', 'error', 'select of database failed');
13618: 
13619: 			$d = fetch_assoc($q);
13620: 			if (empty($d['id_user'])) return $this->showError('404', 'error', 'driver not found');
13621: 			
13622: 			$q = query("BEGIN");
13623: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
13624: 
13625: 			$s = "INSERT IGNORE INTO `order_driver_select`
13626: 					SET
13627: 						`id_order` = '" . $id_order . "',
13628: 						`id_user` = '" . $id_user . "',
13629: 						`create_datetime` = now()
13630: 			";
13631: 
13632: 			$q = query($s);
13633: 	
13634: 			global $link;
13635: 			if (mysqli_affected_rows($link) === -1) 
13636: 			{
13637: 				return $this->showError('404', 'error', 'database insert failed');
13638: 			}
13639: 			elseif (mysqli_affected_rows($link) === 0) 
13640: 			{
13641: 				return $this->showError('404', 'error', 'offered booking');
13642: 			}
13643: 			
13644: 			if (!empty($trips))
13645: 			{
13646: 				$s = "SELECT
13647: 						COUNT(`id_trip`) as trips_count
13648: 					FROM `trip` 		
13649: 					WHERE	
13650: 						`id_trip` in (" . $trips . ") AND `driver` = '" . $id_user . "'
13651: 					";
13652: 
13653: 				$q = query($s);
13654: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
13655: 
13656: 				$d = fetch_assoc($q);
13657: 				$trips = explode(',', $trips);
13658: 				if ($d['trips_count'] != count($trips)) return $this->showError('404', 'error', 'driver is not trip author');
13659: 				
13660: 				$s = array();
13661: 				foreach ($trips as $id_trip)
13662: 				{
13663: 					$s[] = "('" . $id_order . "', '" . $id_trip . "', now(), now())";
13664: 				}
13665: 				$s = "INSERT INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`, `offer_order_datetime`) VALUES " . implode(",", $s) . "ON DUPLICATE KEY UPDATE `offer_order_datetime` = IF(`offer_order_datetime` = 0,now(),`offer_order_datetime`)";
13666: 
13667: 				$q = query($s);
13668: 				if ($q === false) return $this->showError('404', 'error', 'insert in database failed');
13669: 			}
13670: 	
13671: 			$s = "UPDATE `order`
13672: 				SET
13673: 					`last_edit_datetime` = now(),
13674: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
13675: 				WHERE
13676: 					`id_order` = '" . $id_order . "'
13677: 				";
13678: 
13679: 			$q = query($s);
13680: 			if ($q === false) return $this->showError('404', 'error', 'database timestamp update failed');
13681: 
13682: 			$q = query("COMMIT");
13683: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
13684: 	
13685: 			return array(
13686: 				'code' 		=>	'200',
13687: 				'status' 	=>	'success'
13688: 			);
13689: 		}
```

### `queryString` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `17884-18047`
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
18047: 		}
```

### `queryTemplate` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20143-20253`
```php
20143: 		public function queryTemplate($data = "", $template = "", $var = NULL, $sql_templates = array())
20144: 		{
20145: 			if (empty($_SESSION[UID])) {
20146: 				return $this->showError('404', 'error', 'unauthorized access');
20147: 			}			
20148: 
20149: 			if (empty($template)) return $this->showError('404', 'error', 'empty template');
20150: 			
20151: 			if (!isset($sql_templates[$template])) return $this->showError('404', 'error', 'template not found');
20152: 
20153: 			if (!empty($sql_templates[$template]['only_admin']) && $this->id_role != 4)
20154: 			{
20155: 				return $this->showError('404', 'error', 'not enough rights');
20156: 			}
20157: 			$statement_arr = empty($sql_templates[$template]['statement']) ? array() : $sql_templates[$template]['statement'];
20158: 
20159: 			if (empty($sql_templates[$template]['value']['code'])) return $this->showError('404', 'error', 'wrong template');
20160: 			$s = $sql_templates[$template]['value']['code'];
20161: 
20162: 			if (count($statement_arr))
20163: 			{
20164: 				while (true)
20165: 				{
20166: 					foreach($statement_arr as $statement)
20167: 					{
20168: 						if (trim(strtolower(substr($s,0,strlen($statement)+1))) === $statement) break 2;
20169: 					}
20170: 					return $this->showError('404', 'error', 'wrong template statement');
20171: 					break;
20172: 				}
20173: 			}
20174: 
20175: 			if (!empty($data))
20176: 			{
20177: 				$data = json_decode($data,true);
20178: 
20179: 				if (empty($data) || !is_array($data)) 
20180: 				{
20181: 					return $this->showError('404', 'error', 'wrong data');
20182: 				}
20183: 				foreach($data as $d_var=>$val)
20184: 				{		
20185: 					$s = str_replace($d_var,real_escape_string($val),$s);
20186: 				}
20187: 				$sys_data = array(
20188: 									'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
20189: 									'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
20190: 									'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
20191: 									'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
20192: 									'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
20193: 									'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
20194: 									'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
20195: 									'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
20196: 									'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
20197: 									'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
20198: 									'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
20199: 									'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
20200: 									'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
20201: 									'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
20202: 									'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
20203: 									'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
20204: 				);
20205: 				foreach($sys_data as $d_var=>$val)
20206: 				{		
20207: 					$s = str_replace($d_var,real_escape_string($val),$s);
20208: 				}
20209: 			}
20210: 
20211: 			if ($var !== NULL && $var !== "")
20212: 			{
20213: 				$q = query("BEGIN");
20214: 				if ($q === false) return $this->showError('404', 'error', error_db());
20215: 			}
20216: 			$q = query($s);
20217: 			if ($q === false) return $this->showError('404', 'error', error_db());
20218: 
20219: 			$out = array();
20220: 			if ($q !== true)
20221: 			{
20222: 				while ($d = fetch_assoc($q))
20223: 				{
20224: 					$out[] = $d;
20225: 				}
20226: 			}
20227: 			else		
20228: 			{
20229: 				$out['id'] = insert_id();
20230: 				global $link;
20231: 				$out['rows'] = mysqli_affected_rows($link);
20232: 			}
20233: 
20234: 			if ($var !== NULL && $var !== "")
20235: 			{
20236: 				if ($var[0] !== '@') $var = "@$var";
20237: 				$q = query("SELECT $var");
20238: 				if ($q === false) return $this->showError('404', 'error', error_db());
20239: 				$d = fetch_assoc($q);
20240: 				if ($d !== NULL && array_key_exists($var,$d))
20241: 				{
20242: 					$out[] = array('{is_var}'=>1,'var'=>$d[$var]);
20243: 				}
20244: 				$q = query("COMMIT");
20245: 				if ($q === false) return $this->showError('404', 'error', error_db());
20246: 			}		
20247: 
20248: 			return array(
20249: 				'code' 		=>	'200',
20250: 				'status' 	=>	'success',
20251: 				'data'		=>	$out
20252: 			);
20253: 		}
```

### `rateOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `7396-7524`
```php
7396: 		public function rateOrder($id_order = "", $rating = "")
7397: 		{
7398: 			if (empty($_SESSION[UID])) {
7399: 				return $this->showError('404', 'error', 'unauthorized access');
7400: 			}
7401: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
7402: 			{
7403: 				return $this->showError('404', 'error', 'wrong user role');
7404: 			}
7405: 
7406: 			if ($this->id_role == 1 || $this->id_role == 5)
7407: 			{
7408: 				$s = "SELECT
7409: 						`id_order`,
7410: 						`client`,
7411: 						`id_order_status`,
7412: 						`rating`
7413: 					FROM `order` 		
7414: 					WHERE	
7415: 						`id_order` = '" . $id_order . "'
7416: 					LIMIT 1
7417: 					";
7418: 
7419: 				$q = query($s);
7420: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7421: 
7422: 				$d = fetch_assoc($q);
7423: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7424: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
7425: 				if ($d['client'] != $_SESSION[UID]) 
7426: 				{
7427: 					return $this->showError('404', 'error', 'user is not author');
7428: 				}
7429: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
7430: 
7431: 				$s = "UPDATE `order`
7432: 					SET 
7433: 						`rating` = '" . $rating  . "',
7434: 						`last_edit_datetime` = now(),
7435: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
7436: 					WHERE
7437: 						`id_order` = '" . $id_order . "' AND `rating` IS NULL
7438: 					";
7439: 
7440: 				query($s);
7441: 		
7442: 				global $link;
7443: 				if (mysqli_affected_rows($link) === -1)
7444: 				{
7445: 					return $this->showError('404', 'error', 'database update failed');
7446: 				}
7447: 				elseif (mysqli_affected_rows($link) === 0)
7448: 				{
7449: 
7450: 					return $this->showError('404', 'error', 'modified data not found');
7451: 				}
7452: 			}
7453: 			else
7454: 			{
7455: 				$s = "SELECT
7456: 						`order`.`id_order`,
7457: 						`order`.`id_order_status`,
7458: 						od.`id_user`,
7459: 						od.`id_order_driver_status`,
7460: 						od.`rating`
7461: 					FROM `order`
7462: 					LEFT JOIN (
7463: 							SELECT
7464: 								`id_order`,
7465: 								`id_user`,
7466: 								`id_order_driver_status`,
7467: 								`rating`
7468: 							FROM
7469: 								`order_driver`
7470: 							WHERE
7471: 								`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
7472: 						) od USING (`id_order`)				
7473: 					WHERE	
7474: 						`order`.`id_order` = '" . $id_order . "'
7475: 					LIMIT 1
7476: 					";
7477: 
7478: 				$q = query($s);
7479: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7480: 
7481: 				$d = fetch_assoc($q);
7482: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7483: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
7484: 				if (empty($d['id_user'])) 
7485: 				{
7486: 					return $this->showError('404', 'error', 'user is not performer');
7487: 				}
7488: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
7489: 				if ($d['id_order_driver_status'] == 1 || $d['id_order_driver_status'] == 2) 
7490: 				{
7491: 					return $this->showError('404', 'error', 'wrong booking driver state');
7492: 				}
7493: 
7494: 				$s = "UPDATE `order`,`order_driver`
7495: 					SET 
7496: 						`order`.`last_edit_datetime` = now(),
7497: 						`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7498: 						`order_driver`.`rating` = '" . $rating  . "'
7499: 					WHERE
7500: 						`order`.`id_order` = '" . $id_order . "' AND
7501: 						`order`.`id_order` = `order_driver`.`id_order` AND 
7502: 						`order_driver`.`id_order` = '" . $id_order . "' AND 
7503: 						`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7504: 						`order_driver`.`rating` IS NULL
7505: 					";
7506: 
7507: 				query($s);
7508: 
7509: 				global $link;
7510: 				if (mysqli_affected_rows($link) === -1) 
7511: 				{
7512: 					return $this->showError('404', 'error', 'driver update failed');
7513: 				}
7514: 				elseif (mysqli_affected_rows($link) === 0) 
7515: 				{
7516: 					return $this->showError('404', 'error', 'driver modified data not found');
7517: 				}
7518: 			}
7519: 
7520: 			return array(
7521: 				'code' 		=>	'200',
7522: 				'status' 	=>	'success'
7523: 			);
7524: 		}
```

### `readTicket` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `18233-18608`
```php
18233: 		public function readTicket($id_trip = "", $seat = "", $pdf = false, $lang_vls = array(), $price_time_functions = array(), $s_data_stt = array(), $aggregators = array())
18234: 		{
18235: 			if (empty($_SESSION[UID])) {
18236: 				return $this->showError('404', 'error', 'unauthorized access');
18237: 			}
18238: 
18239: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
18240: 			
18241: 			if (empty($seat))
18242: 			{
18243: 				$s = "SELECT			
18244: 						`trip`.`id_trip`,
18245: 						`trip`.`driver`,
18246: 						`trip`.`json`,
18247: 						GROUP_CONCAT(`ticket`.`id_seat` SEPARATOR ',') as seats
18248: 					FROM `trip`
18249: 					LEFT JOIN `ticket` ON `ticket`.`id_trip` = `trip`.`id_trip` AND `ticket`.`blob_link` != ''
18250: 					WHERE	
18251: 						`trip`.`id_trip` = '" . $id_trip . "'
18252: 					LIMIT 1
18253: 					";
18254: 				
18255: 				$q = query($s);
18256: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
18257: 
18258: 				$d = fetch_assoc($q);
18259: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18260: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18261: 				{
18262: 					return $this->showError('404', 'error', 'user is not author');
18263: 				}
18264: 
18265: 				$t_options = json_decode($d['json'],true);
18266: 				if (empty($t_options['seats_sold']))
18267: 				{
18268: 					$seats = empty($d['seats']) ? array() : explode(',',$d['seats']);
18269: 				}
18270: 				else
18271: 				{
18272: 					$folder = CONFIG_USER_FILE_PATH . "trips/$id_trip/ticket/";
18273: 					$seats = array();				
18274: 					if ($dir = scandir($folder))
18275: 					{
18276: 						foreach($dir as $obj)
18277: 						{
18278: 							if ($obj != "." && $obj != ".." && is_file("$folder$obj"))
18279: 							{
18280: 								$seats[] = $obj;
18281: 							}
18282: 						}
18283: 					}
18284: 				}
18285: 
18286: 				return array(
18287: 					'code' 		=>	'200',
18288: 					'status' 	=>	'success',
18289: 					'data'		=>	array('seats' => $seats)
18290: 				);
18291: 			}
18292: 			else
18293: 			{
18294: 				$sql_field = $sql_left_join = '';
18295: 				if ($pdf === true)
18296: 				{
18297: 					$sql_field = ",
18298: 						`order`.`options`,
18299: 						`order`.`sum`,
18300: 						`order`.`currency` as sum_currency,
18301: 						`ticket`.`id_order`,
18302: 						`ticket`.`number`,
18303: 						`ticket`.`code`,
18304: 						`ticket`.`code_qr_base64`,
18305: 						`ticket`.`tariff` as ti_tariff,
18306: 						`ticket`.`currency` as ti_currency,
18307: 						`schedule`.`id_schedule`,
18308: 						`schedule`.`team1`,
18309: 						`schedule`.`team2`,
18310: 						`schedule`.`id_stadium` as stadium,
18311: 						`schedule`.`id_tournament` as tournament,
18312: 						`schedule`.`start_datetime` as datetime,
18313: 						`schedule`.`only_date`,
18314: 						`schedule`.`time_zone`,
18315: 						`schedule`.`code_ean_base64`,						
18316: 						`schedule`.`id_price_time_function` as sc_price_time_function,
18317: 						`schedule`.`currency` as sc_currency,
18318: 						`schedule`.`currency_priority` as sc_currency_priority,
18319: 						`schedule`.`fee` as sc_fee,
18320: 						`schedule`.`tariff` as sc_tariff,
18321: 						`schedule`.`tariff_priority` as sc_tariff_priority,						
18322: 						`trip`.`id_price_time_function` as price_time_function,
18323: 						`trip`.`currency`,
18324: 						`trip`.`currency_priority`,
18325: 						`trip`.`fee`,
18326: 						`trip`.`tariff`,
18327: 						`trip`.`tariff_priority`,
18328: 						`trip`.`create_datetime`,
18329: 						`trip`.`id_aggregator` as ag_id,
18330: 						`users`.`email`,
18331: 						`users`.`name`,
18332: 						`users`.`family`,
18333: 						`users`.`middle`";
18334: 					$sql_left_join = "LEFT JOIN `schedule` ON `schedule`.`id_schedule` = `trip`.`id_schedule`
18335: 					LEFT JOIN `users` ON `users`.`id_user` = `order`.`client`";
18336: 				}
18337: 
18338: 				$s = "SELECT			
18339: 						`trip`.`id_trip`,
18340: 						`trip`.`driver`,
18341: 						`trip`.`json`,
18342: 						`ticket`.`blob_link`,
18343: 						`ticket`.`blob_mime`,
18344: 						`ticket`.`blob_ext_w._dot`,
18345: 						`ticket`.`id_seat`,
18346: 						`ticket`.`id_trip_seat`,
18347: 						`order`.`client`,
18348: 						`order`.`pay_datetime`$sql_field
18349: 					FROM `trip`
18350: 					LEFT JOIN `ticket` ON `ticket`.`id_trip` = `trip`.`id_trip` AND `ticket`.`id_seat` = '" . $seat . "'
18351: 					LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`
18352: 					$sql_left_join
18353: 					WHERE	
18354: 						`trip`.`id_trip` = '" . $id_trip . "'
18355: 					LIMIT 1
18356: 					";
18357: 				
18358: 				$q = query($s);
18359: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
18360: 
18361: 				$d = fetch_assoc($q);
18362: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18363: 
18364: 				$t_options = json_decode($d['json'],true);
18365: 				if (empty($t_options['seats_sold']))
18366: 				{
18367: 					if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
18368: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18369: 					{
18370: 						if (empty($d['client']) || $d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller or buyer');
18371: 						if ($d['pay_datetime'] == '0000-00-00 00:00:00') return $this->showError('404', 'error', 'unpaid order');
18372: 					}
18373: 				}
18374: 				else
18375: 				{
18376: 					if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
18377: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18378: 					{
18379: 						if (!array($t_options['seats_sold'][$seat]) || count($t_options['seats_sold'][$seat]) < 2 || $t_options['seats_sold'][$seat][1] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller and buyer');
18380: 					}
18381: 				}
18382: 				$blob_link = 0;
18383: 				if ($d['blob_link'] === '') 
18384: 				{
18385: 					if ($pdf === true)
18386: 					{
18387: 						$blob_link = 1;
18388: 					}
18389: 					else
18390: 					{
18391: 						return $this->showError('404', 'error', 'empty ticket file link');
18392: 					}
18393: 				}
18394: 
18395: 				if ($pdf === true)
18396: 				{
18397: 					$sc_id = $d['id_schedule'];
18398: 
18399: 					$lang = isset($_SESSION['lang']) ? $_SESSION['lang'] : DEFAULT_LANG;
18400: 					$lang_name = taxi::$data['langs'][$lang]['iso'];
18401: 
18402: 					$html_pdf_ticket_paid_body = 'html_pdf_ticket_paid_body';
18403: 					foreach(array($html_pdf_ticket_paid_body) as $lang_vls_name_base)
18404: 					{
18405: 						foreach(array("_$sc_id","_tournament_{$d['tournament']}") as $suffix)		 
18406: 						{
18407: 							$lang_vls_name = "$lang_vls_name_base$suffix";
18408: 							if (isset($lang_vls[$lang_vls_name][$lang]) && substr($lang_vls[$lang_vls_name][$lang],0,1) !== chr(0))
18409: 							{
18410: 								$$lang_vls_name_base = $lang_vls_name;
18411: 								break;
18412: 							}
18413: 						}
18414: 					}
18415: 					$pdf = lang($html_pdf_ticket_paid_body);
18416: 					if (empty($pdf)) return $this->showError('404', 'error', 'empty template');
18417: 
18418: 					if(!empty($d['options'])) $d['options'] = json_decode($d['options'],true);
18419: 					
18420: 					$seat_s_b_r_s = explode(';',$d['id_seat']);							
18421: 					$ticket = $seats = array();
18422: 
18423: 					$format = $this->constant['email_date_format'];
18424: 					preg_match('/[eIOPpTZcrU]/',$format ,$format_with_tz);
18425: 					add_time_zone($d['datetime']);
18426: 					add_time_zone($d['create_datetime']);
18427: 
18428: 					if (isset($d['options']['tickets']['seats'][$id_trip][$d['id_trip_seat']]))
18429: 					{
18430: 						$price = $d['options']['tickets']['seats'][$id_trip][$d['id_trip_seat']];
18431: 					}
18432: 					else
18433: 					{
18434: 						if (!empty($d['datetime']))
18435: 						{
18436: 							$ptfd = $this->createPriceTimeFunctionData(array(
18437: 								'price_time_function'		=>	$d['price_time_function'],
18438: 								'sc_price_time_function'	=>	$d['sc_price_time_function'],
18439: 								't_create_datetime'			=>	$d['create_datetime'],
18440: 								'sc_datetime'				=>	$d['datetime']
18441: 							),$price_time_functions);
18442: 						}
18443: 						$currency = $currency_priority = NULL;
18444: 						if (isset($d['currency']))
18445: 						{
18446: 							$currency = $d['currency'];
18447: 						}
18448: 						elseif (isset($d['sc_currency']))
18449: 						{
18450: 							$currency = $d['sc_currency'];
18451: 						}
18452: 						if (isset($currency))
18453: 						{
```

### `registerUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `15-336`
```php
15: 		public function registerUser($role = 0, $phone = "", $email = "", $tg = "", $wa = "", $name = "", $referral_code = "", $reco = "", $ip = "", $data = "", $show_token = false, $roles = array())
16: 		{
17: 			$sql_user = "NULL";
18: 			$referrer = "";
19: 			if (empty($tg) && empty($wa))
20: 			{
21: 				if (empty($_SESSION[UID])) 
22: 				{
23: 					if ($role != 1 && $role != 2 && $role != 5)
24: 					{
25: 						return $this->showError('404', 'error', 'wrong user role');
26: 					}
27: 				}
28: 				else
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
32: 					$sql_user = "'" . $_SESSION[UID] . "'";
33: 				}
34: 			}
35: 			else
36: 			{
37: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
38: 				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
39: 				$sql_user = "'" . $_SESSION[UID] . "'";
40: 			}
41: 
42: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa)) 
43: 			{
44: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
45: 			} 
46: 			else
47: 			{
48: 				$sql_email = "`email` = '" . $email . "'";
49: 				$sql_phone = "`phone` = '" . preg_replace('/[^0-9]+/','',$phone) . "'";
50: 				$sql_tg = "`tg` = '" . $tg . "'";
51: 				$sql_wa = "`wa` = '" . $wa . "'";				
52: 				
53: 				$sql_field = array();
54: 				$sql_where = array();
55: 				if (empty($phone)) 
56: 				{
57: 					$sql_phone = "`phone` = NULL";
58: 				}
59: 				else
60: 				{
61: 					$sql_field[] = "COUNT(IF(" . $sql_phone . ",1,NULL)) as pflag";
62: 					$sql_where[] = $sql_phone;
63: 				}
64: 				if (empty($email)) 
65: 				{
66: 					$sql_email = "`email` = NULL";
67: 				}
68: 				else
69: 				{
70: 					$sql_field[] = "COUNT(IF(" . $sql_email . ",1,NULL)) as eflag";
71: 					$sql_where[] = $sql_email;
72: 				}
73: 				if (empty($tg)) 
74: 				{
75: 					$sql_tg = "`tg` = NULL";
76: 				}
77: 				else
78: 				{
79: 					$sql_field[] = "COUNT(IF(" . $sql_tg . ",1,NULL)) as tflag";
80: 					$sql_where[] = $sql_tg;
81: 				}
82: 				if (empty($wa)) 
83: 				{
84: 					$sql_wa = "`wa` = NULL";
85: 				}
86: 				else
87: 				{
88: 					$sql_field[] = "COUNT(IF(" . $sql_wa . ",1,NULL)) as wflag";
89: 					$sql_where[] = $sql_wa;
90: 				}
91: 				$sql_field = implode(",\n", $sql_field);
92: 				$sql_where = implode(" OR ", $sql_where);
93: 			}
94: 
95: 			if (!empty($referral_code)) 
96: 			{
97: 				$s = "SELECT 
98: 						`id_user` 
99: 					FROM `users`
100: 					WHERE `referral_code` = '" . $referral_code . "'
101: 					LIMIT 1
102: 					";
103: 
104: 				$q = query($s);
105: 				if ($q === false) return $this->showError('404', 'error', 'ref_code database select failed');
106: 				$d = fetch_assoc($q);			
107: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'wrong ref_code');
108: 				$referrer = $d['id_user'];
109: 			}
110: 			elseif(!empty($reco))
111: 			{
112: 				$s = "SELECT 
113: 						`id_user` 
114: 					FROM `users`
115: 					WHERE `id_user` = '" . $reco . "'
116: 					LIMIT 1
117: 					";
118: 
119: 				$q = query($s);
120: 				if ($q === false) return $this->showError('404', 'error', 'ref_code database select failed');
121: 				$d = fetch_assoc($q);
122: 				if (!empty($d['id_user'])) $referrer = $reco;
123: 			}
124: 			if (empty($referrer))
125: 			{
126: 					$s = "SELECT 
127: 							`id_user`
128: 						FROM `ip_referral`
129: 						WHERE 
130: 							`ip` = INET_ATON('" . $ip . "')
131: 						LIMIT 1
132: 						";
133: 					$q = query($s);
134: 					if ($q === false) return $this->showError('404', 'error', 'ip database select failed');
135: 					$d = fetch_assoc($q);
136: 					if (!empty($d['id_user'])) $referrer = $d['id_user'];
137: 			}
138: 
139: 			$s = "SELECT 
140: 					" . $sql_field . " 
141: 				FROM `users`
142: 				WHERE " . $sql_where . "
143: 				";
144: 
145: 			$q = query($s);
146: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
147: 			$d = fetch_assoc($q);
148: 			$msg = array();
149: 			if (!empty($d['pflag']))
150: 			{
151: 				$msg[] = 'double phone';
152: 			}
153: 			if (!empty($d['eflag']))
154: 			{
155: 				$msg[] = 'double email';
156: 			}
157: 			if (!empty($d['tflag']))
158: 			{
159: 				$msg[] = 'double tg';
160: 			}
161: 			if (!empty($d['wflag']))
162: 			{
163: 				$msg[] = 'double wa';
164: 			}
165: 			if (!empty($msg))return $this->showError('404', 'error', 'busy user data: ' . implode(", ", $msg));
166: 
167: 			if (preg_match('/^(\S+)(?:\s+(?:(.*?)\s+)?(\S+))?$/',$name,$name_middle_family))
168: 			{
169: 				$name_sql = "`name` = '" . $name_middle_family[1] . "',\n";
170: 				$name_sql .= empty($name_middle_family[2]) ? "" : "`middle` = '" .  $name_middle_family[2] . "',\n";
171: 				$name_sql .= empty($name_middle_family[3]) ? "" : "`family` = '" . $name_middle_family[3] . "',\n";
172: 			}
173: 			else
174: 			{
175: 				$name_sql = "";
176: 			}
177: 
178: 			$sql_data = "";
179: 			if (!empty($data)) 
180: 			{
181: 				$data = json_decode($data,true);
182: 				if (!empty($data))
183: 				{
184: 					if (!is_array($data)) return $this->showError('404', 'error', 'wrong data');
185: 					$allowed_fields = array(
186: 											'u_details'				=>		'json'
187: 					);
188: 
189: 					$forbidden_fields = array();
190: 					$affected_fields = array();
191: 					$affected_keys = array();
192: 
193: 					foreach ($data as $key => $value)
194: 					{
195: 						if (is_string($value)) $data[$key] = trim($value);
196: 
197: 						if (!empty($allowed_fields[$key]))
198: 						{
199: 							$affected_fields[] = $key;
200: 							$affected_keys[$key] = true;				
201: 						}
202: 						else
203: 						{
204: 							$forbidden_fields[] = $key;
205: 						}
206: 					}
207: 
208: 					if (!empty($affected_keys['u_details']))
209: 					{
210: 						if (!empty($this->constant['u_details_valid_keys'])) 
211: 						{
212: 							if (!is_array($data['u_details'])) return $this->showError('404', 'error', 'u_details not array');
213: 
214: 							if (!empty($data['u_details']))
215: 							{
216: 								if (array_includes_only_add_keys_from_assoc_list($data['u_details'],$this->constant['u_details_valid_keys']) === false) return $this->showError('404', 'error', 'wrong u_details keys');
217: 							}
218: 						}
219: 
220: 						$data['u_details'] = real_escape_string(json_encode($data['u_details']));
221: 					}
222: 
223: 					if (!empty($affected_keys))
224: 					{
225: 						$sql_data = array();
226: 						foreach ($affected_fields as $key)
227: 						{
228: 							$sql_data[] = "`" . $allowed_fields[$key] . "` = " 
229: 								 . ($data[$key] === NULL ? "NULL" : "'" . $data[$key] . "'");
230: 						}
231: 						$sql_data = implode(",\n", $sql_data) . ',';
232: 					}
233: 
234: 					if (isset($data['password'])) 
235: 					{
```

### `removeFavoriteUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `10879-10919`
```php
10879: 		public function removeFavoriteUser($id_user = "", $id_favorite = "")
10880: 		{
10881: 			if (empty($_SESSION[UID])) {
10882: 				return $this->showError('404', 'error', 'unauthorized access');
10883: 			}
10884: 
10885: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10886: 			{	
10887: 				$id_user = $_SESSION[UID];
10888: 			}
10889: 			else
10890: 			{	
10891: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10892: 				$s = "SELECT
10893: 						`id_user`
10894: 					FROM `users`
10895: 					WHERE `id_user` = '" . $id_user . "'
10896: 					LIMIT 1
10897: 					";
10898: 
10899: 				$q = query($s);
10900: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10901: 				$d = fetch_assoc($q);
10902: 				
10903: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10904: 			}
10905: 
10906: 			$s = "DELETE
10907: 				FROM `users_favorite`
10908: 				WHERE 
10909: 					`id_user` = '" . $id_user . "' AND `id_favorite` in (" . $id_favorite  . ")
10910: 				";
10911: 			$q = query($s);
10912: 			
10913: 			if ($q === false) return $this->showError('404', 'error', 'database delete failed');
10914: 
10915: 			return array(
10916: 				'code' 		=>	'200',
10917: 				'status' 	=>	'success'
10918: 			);
10919: 		}
```

### `requestTemplate` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20638-20826`
```php
20638: 		public function requestTemplate($template = "", $is_var = false, $outer_script_templates = array())
20639: 		{	
20640: 			if (empty($template)) return $this->showError('404', 'error', 'empty template');
20641: 
20642: 			if (empty($is_var))
20643: 			{
20644: 				if (!isset($outer_script_templates[$template])) return $this->showError('404', 'error', 'template not found');
20645: 				$outer_script_template =  $outer_script_templates[$template];
20646: 			}
20647: 			else
20648: 			{
20649: 				while (true)
20650: 				{
20651: 					foreach($outer_script_templates as $outer_script_template)
20652: 					{
20653: 						if ($outer_script_template['var'] == $template) 
20654: 						{
20655: 							break 2;
20656: 						}
20657: 					}
20658: 					return $this->showError('404', 'error', 'outer script template not found');
20659: 				}
20660: 			}
20661: 
20662: 			switch ($outer_script_template['only_admin'])
20663: 			{
20664: 				case '-1':
20665: 					$u_id_export = isset($_SESSION[UID]) ? $_SESSION[UID] : '';
20666: 					break;
20667: 				case '0':
20668: 					if (empty($_SESSION[UID])) {
20669: 						return $this->showError('404', 'error', 'unauthorized access');
20670: 					}
20671: 					$u_id_export = $_SESSION[UID];
20672: 					break;
20673: 				case '1':
20674: 					if ($this->id_role != 4)
20675: 					{
20676: 						return $this->showError('404', 'error', 'not enough rights');
20677: 					}
20678: 					$u_id_export = $_SESSION[UID];
20679: 					break;
20680: 			}
20681: 
20682: 			$post_json = $outer_script_template['post_json'];
20683: 			$post_json_export = $outer_script_template['post_json_export'];
20684: 			if (empty($post_json) || !is_array($post_json))
20685: 			{
20686: 				$arr = array();
20687: 			}
20688: 			else
20689: 			{
20690: 				$arr = $post_json;
20691: 			}
20692: 			if (!empty($post_json_export) && is_array($post_json_export))
20693: 			{
20694: 				foreach($post_json_export as $key=>$val)
20695: 				{
20696: 					if (isset($_REQUEST[$key]))
20697: 					{
20698: 						if (empty($val) && $val !== '0')
20699: 						{
20700: 							$arr[$key] = $_REQUEST[$key];
20701: 						}
20702: 						elseif (!is_array($val))
20703: 						{
20704: 							$arr[$val] = $_REQUEST[$key];
20705: 						}
20706: 						elseif (empty($val[0]) && (!isset($val[0]) || $val[0] !== '0'))
20707: 						{
20708: 							$arr[$key] = $_REQUEST[$key];
20709: 						}
20710: 						else
20711: 						{
20712: 							if (is_array($val[0]))
20713: 							{
20714: 								$arr = set_val_for_array($arr,$val[0],$_REQUEST[$key]);
20715: 							}
20716: 							else
20717: 							{
20718: 								$arr[$val[0]] = $_REQUEST[$key];
20719: 							}
20720: 						}
20721: 					}
20722: 					elseif (!empty($val) && is_array($val) && isset($val[1]))
20723: 					{
20724: 						if (empty($val[0]) && (!isset($val[0]) || $val[0] !== '0'))
20725: 						{
20726: 							$arr[$key] = $val[1];
20727: 						}
20728: 						elseif (is_array($val[0]))
20729: 						{
20730: 							$arr = set_val_for_array($arr,$val[0],$val[1]);
20731: 						}
20732: 						else
20733: 						{
20734: 							$arr[$val[0]] = $val[1];
20735: 						}
20736: 					}
20737: 				}
20738: 			}
20739: 			if (!empty($outer_script_template['u_id_export'])) $arr['u_id'] = $u_id_export;
20740: 			$headers_json = $outer_script_template['headers_json'];
20741: 			$headers_json_export = $outer_script_template['headers_json_export'];
20742: 			if (empty($headers_json) || !is_array($headers_json))
20743: 			{
20744: 				$headers = array();
20745: 			}
20746: 			else
20747: 			{
20748: 				$headers = $headers_json;
20749: 			}
20750: 			if (empty($outer_script_template['urlencoded']))
20751: 			{
20752: 				$data_str = json_encode($arr);
20753: 				$headers[] = "Content-Type: application/json";
20754: 			}
20755: 			else
20756: 			{
20757: 				$data_str = array();
20758: 				foreach($arr as $key=>$val)
20759: 				{
20760: 					$data_str[] = urlencode($key) . '=' . urlencode(is_array($val) ? json_encode($val) : $val);		 
20761: 				}
20762: 				$data_str = implode('&',$data_str);
20763: 				$headers[] = "Content-Type: application/x-www-form-urlencoded";
20764: 			}
20765: 			
20766: 			$url = $outer_script_template['url'];
20767: 			$c = curl_init();
20768: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
20769: 			curl_setopt($c,CURLOPT_URL, $url);
20770: 			curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
20771: 			curl_setopt($c,CURLOPT_POST, 1);
20772: 			curl_setopt($c,CURLOPT_POSTFIELDS,$data_str);
20773: 			curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
20774: 			if (!empty($headers_json_export) && is_array($headers_json_export))
20775: 			{
20776: 				$h_arr = getallheaders();
20777: 				$h_arr_lower = array();
20778: 				foreach($h_arr as $key=>$val)
20779: 				{
20780: 					$h_arr_lower[strtolower($key)] = $val;
20781: 				}
20782: 				unset($h_arr);
20783: 				foreach($headers_json_export as $h=>$v)
20784: 				{
20785: 					if (isset($h_arr_lower[$h]))
20786: 					{
20787: 						if (empty($v) && $v !== '0')
20788: 						{
20789: 							$headers[] = "$h: {$h_arr_lower[$h]}";
20790: 						}
20791: 						elseif (!is_array($v))
20792: 						{
20793: 							$headers[] = "$v: {$h_arr_lower[$h]}";
20794: 						}
20795: 						elseif (empty($v[0]) && (!isset($v[0]) || $v[0] !== '0'))
20796: 						{
20797: 							$headers[] = "$h: {$h_arr_lower[$h]}";
20798: 						}
20799: 						else
20800: 						{
20801: 							$headers[] = "{$v[0]}: {$h_arr_lower[$h]}";
20802: 						}
20803: 					}
20804: 					elseif (!empty($v) && is_array($v) && isset($v[1]))
20805: 					{
20806: 						if (empty($v[0]) && (!isset($v[0]) || $v[0] !== '0'))
20807: 						{
20808: 							$headers[] = "$h: {$v[1]}";
20809: 						}
20810: 						else
20811: 						{
20812: 							$headers[] = "{$v[0]}: {$v[1]}";
20813: 						}
20814: 					}
20815: 				}
20816: 			}
20817: 			if (!empty($headers)) curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
20818: 			$response = curl_exec($c);
20819: 			curl_close($c);
20820: 
20821: 			return array(
20822: 				'code' 		=>	'200',
20823: 				'status' 	=>	'success',
20824: 				'data'		=>	$response
20825: 			);
20826: 		}
```

### `selectActiveOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `4475-4941`
```php
4475: 		public function selectActiveOrder($fields = 0)
4476: 		{
4477: 			if (empty($_SESSION[UID])) {
4478: 				return $this->showError('404', 'error', 'unauthorized access');
4479: 			}
4480: 
4481: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
4482: 			{
4483: 				return $this->showError('404', 'error', 'wrong user role');
4484: 			}
4485: 
4486: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = $sql_where = '';
4487: 						
4488: 			$field_flag = array();
4489: 			if (!empty($fields))
4490: 			{
4491: 				$field_arr	= get_field_arr('activeOrder',$this->id_role);
4492: 				$bin_arr = get_bin_arr();
4493: 
4494: 				foreach(str_split($fields) as $index => $char)
4495: 				{
4496: 					$value = get_0_64($char);
4497: 					if (empty($value)) continue;
4498: 					foreach($bin_arr as $bin_i)
4499: 					{
4500: 						if ($value & $bin_i) 
4501: 						{
4502: 							if (isset($field_arr[$index][$bin_i]))
4503: 							{
4504: 								$field_flag[$field_arr[$index][$bin_i]] = true;
4505: 							}
4506: 						}
4507: 					}
4508: 				}
4509: 			}
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
4536: 			{
4537: 				$sql_order .= "(SELECT
4538: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
4539: 						 FROM
4540: 							`ticket`
4541: 						 WHERE `ticket`.`id_order` = `order`.`id_order`
4542: 						) as fix_seats,";
4543: 			}
4544: 
4545: 			if ($this->id_role == 1 || $this->id_role == 5)
4546: 			{
4547: 				$s = "SELECT
4548: 						`order`.`id_order` as b_id,
4549: 						" . $sql_order . "
4550: 						`order`.`client` as u_id,
4551: 						`order`.`from` as b_start_address,
4552: 						`order`.`from_lat` as b_start_latitude,
4553: 						`order`.`from_lng` as b_start_longitude,
4554: 						`order`.`to` as b_destination_address,
4555: 						`order`.`to_lat` as b_destination_latitude,
4556: 						`order`.`to_lng` as b_destination_longitude,
4557: 						`order`.`datetime_start_plan` as b_start_datetime,
4558: 						`order`.`comment` as b_custom_comment,
4559: 						`order`.`flight_number` as b_flight_number,
4560: 						`order`.`terminal` as b_terminal,
4561: 						`order`.`passenger_count` as b_passengers_count,
4562: 						`order`.`luggage_count` as b_luggage_count,
4563: 						`order`.`placard` as b_placard,
4564: 						`order`.`id_car_class` as b_car_class,
4565: 						`order`.`id_order_status` as b_state,
4566: 						`order`.`create_datetime` as b_created,
4567: 						`order`.`is_confirmed` as b_confirm_state,
4568: 						`order`.`car_count` as b_cars_count,
4569: 						`order`.`approve_datetime` as b_approved,
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
4596: 						) as drivers,
4597: 						(SELECT
4598: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4599: 						 FROM `order_comment_items`
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
4614: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
4615: 					";
4616: 			}
4617: 			else
4618: 			{
4619: 				$s = "SELECT
4620: 						`order`.`id_order` as b_id,
4621: 						" . $sql_order . "
4622: 						`order`.`client` as u_id,
4623: 						`order`.`from` as b_start_address,
4624: 						`order`.`from_lat` as b_start_latitude,
4625: 						`order`.`from_lng` as b_start_longitude,
4626: 						`order`.`to` as b_destination_address,
4627: 						`order`.`to_lat` as b_destination_latitude,
4628: 						`order`.`to_lng` as b_destination_longitude,
4629: 						`order`.`datetime_start_plan` as b_start_datetime,
4630: 						`order`.`comment` as b_custom_comment,
4631: 						`order`.`flight_number` as b_flight_number,
4632: 						`order`.`terminal` as b_terminal,
4633: 						`order`.`passenger_count` as b_passengers_count,
4634: 						`order`.`luggage_count` as b_luggage_count,
4635: 						`order`.`placard` as b_placard,
4636: 						`order`.`id_car_class` as b_car_class,
4637: 						`order`.`id_order_status` as b_state,
4638: 						`order`.`create_datetime` as b_created,
4639: 						`order`.`is_confirmed` as b_confirm_state,
4640: 						`order`.`car_count` as b_cars_count,
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
4669: 						(SELECT
4670: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
4671: 						 FROM `order_comment_items`
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

### `selectArchiveOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `5603-6178`
```php
5603: 		public function selectArchiveOrder($fields = 0)
5604: 		{
5605: 			if (empty($_SESSION[UID])) {
5606: 				return $this->showError('404', 'error', 'unauthorized access');
5607: 			}
5608: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
5609: 			{
5610: 				return $this->showError('404', 'error', 'wrong user role');
5611: 			}
5612: 
5613: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = $sql_where = '';
5614: 						
5615: 			$field_flag = array();
5616: 			if (!empty($fields))
5617: 			{
5618: 				$field_arr	= get_field_arr('archiveOrder',$this->id_role);
5619: 				$bin_arr = get_bin_arr();
5620: 
5621: 				foreach(str_split($fields) as $index => $char)
5622: 				{
5623: 					$value = get_0_64($char);
5624: 					if (empty($value)) continue;
5625: 					foreach($bin_arr as $bin_i)
5626: 					{
5627: 						if ($value & $bin_i) 
5628: 						{
5629: 							if (isset($field_arr[$index][$bin_i]))
5630: 							{
5631: 								$field_flag[$field_arr[$index][$bin_i]] = true;
5632: 							}
5633: 						}
5634: 					}
5635: 				}
5636: 			}
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
5663: 			{
5664: 				$sql_order .= "(SELECT
5665: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
5666: 						 FROM
5667: 							`ticket`
5668: 						 WHERE `ticket`.`id_order` = `order`.`id_order`
5669: 						) as fix_seats,";
5670: 			}
5671: 		
5672: 			if ($this->id_role == 1 || $this->id_role == 5)
5673: 			{
5674: 				$s = "SELECT
5675: 						`order`.`id_order` as b_id,
5676: 						" . $sql_order . "
5677: 						`order`.`client` as u_id,
5678: 						`order`.`from` as b_start_address,
5679: 						`order`.`from_lat` as b_start_latitude,
5680: 						`order`.`from_lng` as b_start_longitude,
5681: 						`order`.`to` as b_destination_address,
5682: 						`order`.`to_lat` as b_destination_latitude,
5683: 						`order`.`to_lng` as b_destination_longitude,
5684: 						`order`.`datetime_start_plan` as b_start_datetime,
5685: 						`order`.`comment` as b_custom_comment,
5686: 						`order`.`flight_number` as b_flight_number,
5687: 						`order`.`terminal` as b_terminal,
5688: 						`order`.`passenger_count` as b_passengers_count,
5689: 						`order`.`luggage_count` as b_luggage_count,
5690: 						`order`.`placard` as b_placard,
5691: 						`order`.`id_car_class` as b_car_class,
5692: 						`order`.`id_payment_method` as b_payment_way,
5693: 						`order`.`id_payment_card` as b_payment_card,
5694: 						`order`.`tips` as b_tips,
5695: 						`order`.`id_order_status` as b_state,
5696: 						`order`.`rating` as b_rating,
5697: 						`order`.`create_datetime` as b_created,
5698: 						`order`.`is_confirmed` as b_confirm_state,
5699: 						`order`.`confirm_limit_datetime` as b_confirmation_limit,
5700: 						`order`.`confirm_datetime` as b_confirmation_datetime,
5701: 						`order`.`sum` as b_payment_sum,
5702: 						`order`.`pay_datetime` as b_payment_datetime,
5703: 						`order`.`car_count` as b_cars_count,
5704: 						`order`.`cancel_reason` as b_cancel_reason,
5705: 						(SELECT
5706: 							GROUP_CONCAT(
5707: 								CONCAT_WS('|',
5708: 									`id_user`,
5709: 									`id_order_cancel`
5710: 								)
5711: 							SEPARATOR ';')
5712: 							FROM `order_cancel_items`
5713: 							WHERE `order_cancel_items`.`id_order` = `order`.`id_order`
5714: 							ORDER BY `id_user`
5715: 						) as b_cancel_states,
5716: 						`order`.`approve_datetime` as b_approved,
5717: 						`order`.`cancel_datetime` as b_canceled,
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
5743: 									`order_driver`.`options`
5744: 								)
5745: 							SEPARATOR 0x01)
5746: 						) as drivers,
5747: 						(SELECT
5748: 							GROUP_CONCAT(`order_comment_items`.`id_order_comment` SEPARATOR ',')
5749: 						 FROM `order_comment_items`
5750: 						 WHERE `order_comment_items`.`id_order` = `order`.`id_order`
5751: 						) as b_comments,
5752: 						(SELECT
5753: 							GROUP_CONCAT(`order_service`.`id_service` SEPARATOR ',')
5754: 						 FROM `order_service`
5755: 						 WHERE `order_service`.`id_order` = `order`.`id_order`
5756: 						) as b_services
5757: 					FROM `order` 
5758: 					LEFT JOIN `order_driver` USING (`id_order`)			
5759: 					WHERE	
5760: 						`order`.`client` = '" . $_SESSION[UID] . "' AND `order`.`id_order_status` in (3,4)
5761: 					GROUP BY `order`.`id_order`
5762: 					ORDER BY `order`.`last_edit_datetime` DESC
5763: 					LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
5764: 					";
5765: 			}
5766: 			else
5767: 			{
5768: 				$s = "SELECT
5769: 						`order`.`id_order` as b_id,
5770: 						" . $sql_order . "
5771: 						`order`.`client` as u_id,
5772: 						`order`.`from` as b_start_address,
5773: 						`order`.`from_lat` as b_start_latitude,
5774: 						`order`.`from_lng` as b_start_longitude,
5775: 						`order`.`to` as b_destination_address,
5776: 						`order`.`to_lat` as b_destination_latitude,
5777: 						`order`.`to_lng` as b_destination_longitude,
5778: 						`order`.`datetime_start_plan` as b_start_datetime,
5779: 						`order`.`comment` as b_custom_comment,
5780: 						`order`.`flight_number` as b_flight_number,
5781: 						`order`.`terminal` as b_terminal,
5782: 						`order`.`passenger_count` as b_passengers_count,
5783: 						`order`.`luggage_count` as b_luggage_count,
5784: 						`order`.`placard` as b_placard,
5785: 						`order`.`id_car_class` as b_car_class,
5786: 						`order`.`tips` as b_tips,
5787: 						`order`.`id_order_status` as b_state,
5788: 						`order`.`rating` as b_rating,
5789: 						`order`.`create_datetime` as b_created,
5790: 						`order`.`is_confirmed` as b_confirm_state,					
5791: 						`order`.`car_count` as b_cars_count,
5792: 						`order`.`cancel_reason` as b_cancel_reason,
5793: 						(SELECT
5794: 							GROUP_CONCAT(
5795: 								CONCAT_WS('|',
5796: 									`id_user`,
5797: 									`id_order_cancel`
5798: 								)
5799: 							SEPARATOR ';')
5800: 							FROM `order_cancel_items`
5801: 							WHERE `order_cancel_items`.`id_order` = `order`.`id_order`
5802: 							ORDER BY `id_user`
5803: 						) as b_cancel_states,
5804: 						`order`.`approve_datetime` as b_approved,
5805: 						`order`.`cancel_datetime` as b_canceled,
5806: 						`order`.`complete_datetime` as b_completed,
5807: 						`order`.`max_waiting_datetime` as b_max_waiting,
5808: 						`order`.`estimated_waiting_datetime` as b_estimate_waiting,
5809: 						`order`.`options` as b_options,
5810: 						`order`.`id_order_location` as b_location_class,
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
5823: 									IFNULL(`order_driver`.`sum`,0x02),
```

### `selectCar` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `1851-2064`
```php
1851: 		public function selectCar($id_car = "", $id_user = "", $langs = array())
1852: 		{
1853: 			if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
1854: 			$sql = $sql_json = $sql_left_join = "";
1855: 			if (!empty($id_user))
1856: 			{
1857: 				if ($id_user == 'authorized') 
1858: 				{
1859: 					if (empty($id_car))
1860: 					{
1861: 						$sql = "";
1862: 					}
1863: 					elseif ($id_car == 'driven')
1864: 					{
1865: 						$sql = "AND `used` = '1'";
1866: 					}
1867: 					else
1868: 					{
1869: 						$sql = "AND `id_car` in (" . $id_car . ")";
1870: 					}
1871: 					$sql_left_join = "
1872: 						LEFT JOIN (
1873: 								SELECT
1874: 									`id_car`	
1875: 								FROM
1876: 									`car_users`
1877: 								WHERE
1878: 									`id_user` = '" . $_SESSION[UID] . "' " . $sql . "
1879: 								GROUP BY
1880: 
1881: 									`id_car`
1882: 							) cu USING (`id_car`)";
1883: 				}
1884: 				else
1885: 				{
1886: 					if (!empty($id_car))
1887: 					{
1888: 						$sql = "AND `id_car` in (" . $id_car . ")";
1889: 					}
1890: 					$sql_left_join = "
1891: 						LEFT JOIN (
1892: 								SELECT
1893: 									`id_car`	
1894: 								FROM
1895: 									`car_users`
1896: 								WHERE
1897: 									`id_user` in (" . $id_user . ") " . $sql . "
1898: 								GROUP BY
1899: 									`id_car`
1900: 							) cu USING (`id_car`)";
1901: 				}
1902: 				$sql = " AND cu.`id_car` IS NOT NULL";
1903: 			}
1904: 			else
1905: 			{
1906: 				if (!empty($id_car))
1907: 				{	
1908: 					$sql = "AND `car`.`id_car` in (" . $id_car . ")";
1909: 				}
1910: 			}
1911: 
1912: 			$sql_json = ",`car`.`json` as details";
1913: 
1914: 			$license_keys = array('id');
1915: 			$license_keys_name = array();
1916: 			$license_keys_desc = array();
1917: 			$sql_license = array();
1918: 			$sql_license_add = array();
1919: 			$null_check_keys = array();
1920: 			foreach ($langs as $lang)
1921: 			{
1922: 				$sql_license[] = "IFNULL(`name_{$lang['iso']}`,0x02)";
1923: 				$sql_license_add[] = "`description_{$lang['iso']}`";
1924: 				$license_keys_name[] = $lang['iso'];
1925: 				$license_keys_desc[] = "about_{$lang['iso']}";
1926: 				$null_check_keys[] = $lang['iso'];
1927: 			}	
1928: 			$license_keys = array_merge($license_keys,$license_keys_name,$license_keys_desc,array('active','b_l_c'));
1929: 			$sql_license = implode(',',$sql_license);
1930: 			$sql_license_add = implode(',',$sql_license_add);
1931: 
1932: 			$s = "SELECT
1933: 					`car`.`id_car` as c_id,
1934: 					`car`.`id_car_model` as cm_id,
1935: 					GROUP_CONCAT(`car_users`.`id_user` SEPARATOR ',') as u_id,
1936: 					GROUP_CONCAT(IF(`car_users`.`used` = '1',`car_users`.`id_user`,NULL) SEPARATOR ',') as u_d_id,
1937: 					`car`.`seats`,
1938: 					`car`.`license_plate` as registration_plate,
1939: 					`car`.`id_car_color` as color,
1940: 					`car`.`photo_link` as photo
1941: 					" . $sql_json . ",
1942: 					`car`.`id_car_class` as cc_id,
1943: 					(SELECT
1944: 						IF(`id_car` IS NULL,NULL,
1945: 							GROUP_CONCAT(
1946: 								CONCAT_WS(0x00,
1947: 									`id_taxi_license`,
1948: 									$sql_license,
1949: 									$sql_license_add,
1950: 									`active`,
1951: 									IFNULL((SELECT
1952: 										IF(`id_order_location` IS NULL,NULL,
1953: 											GROUP_CONCAT(
1954: 												CONCAT_WS('|',
1955: 													`id_order_location`,
1956: 													`location_table_column_value`,
1957: 													IFNULL(`tariff`,0x02),
1958: 													IFNULL(`currency`,0x02)
1959: 												)
1960: 											SEPARATOR ';')
1961: 										)
1962: 									 FROM `taxi_license_order_location`
1963: 									 WHERE `id_taxi_license` = `taxi_license`.`id_taxi_license`
1964: 									),0x02)
1965: 								)
1966: 							SEPARATOR 0x01)
1967: 						)
1968: 					 FROM `taxi_license`
1969: 					 WHERE `id_car` = `car`.`id_car`
1970: 					) as licenses
1971: 				FROM `car`
1972: 					" . $sql_left_join . "
1973: 				LEFT JOIN `car_users` USING (`id_car`)
1974: 				WHERE
1975: 					`car_users`.`id_user` IS NOT NULL
1976: 					" . $sql . "
1977: 				GROUP BY
1978: 					`car_users`.`id_car`
1979: 				";
1980: 
1981: 			$q = query($s);
1982: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
1983: 			
1984: 			$out = array('car' => array());
1985: 			while ($d = fetch_assoc($q))
1986: 			{
1987: 				$d['photo'] = $d['photo'] ? url($d['photo'],FILE_PATH) : '';
1988: 				if (!empty($d['u_id'])){
1989: 					$d['u_id'] = explode(',',$d['u_id']);
1990: 					if ($this->id_role != 4 && !in_array($_SESSION[UID],$d['u_id']))
1991: 					{
1992: 						unset($d['details']);
1993: 					}
1994: 				}
1995: 				else 
1996: 				{
1997: 					if ($this->id_role != 4) unset($d['details']);
1998: 				}
1999: 				if (!empty($d['details'])) {$d['details'] = json_decode($d['details'],true);}
2000: 				if (!empty($d['licenses']))
2001: 				{
2002: 					$d['licenses'] = explode(chr(1),$d['licenses']);
2003: 					foreach ($d['licenses'] as $key=>$value)
2004: 					{
2005: 						$d['licenses'][$key] = array();
2006: 						$value = explode(chr(0),$value);
2007: 						foreach($license_keys as $i=>$l_key)
2008: 						{
2009: 							$d['licenses'][$key][$l_key] = $value[$i];
2010: 						}
2011: 						foreach($null_check_keys as $l_key)
2012: 						{
2013: 							if ($d['licenses'][$key][$l_key] == chr(2)) $d['licenses'][$key][$l_key] = NULL;
2014: 						}
2015: 						if ($d['licenses'][$key]['b_l_c'] == chr(2) ) {$d['licenses'][$key]['b_l_c'] = NULL; continue;}
2016: 						$d['licenses'][$key]['b_l_c'] = explode(';',$d['licenses'][$key]['b_l_c']);
2017: 						foreach ($d['licenses'][$key]['b_l_c'] as $blc_key=>$blc_value)
2018: 						{
2019: 							$d['licenses'][$key]['b_l_c'][$blc_key] = array();
2020: 							list(
2021: 								$d['licenses'][$key]['b_l_c'][$blc_key]['location'],
2022: 								$d['licenses'][$key]['b_l_c'][$blc_key]['value'],
2023: 								$d['licenses'][$key]['b_l_c'][$blc_key]['tariff'],
2024: 								$d['licenses'][$key]['b_l_c'][$blc_key]['currency']
2025: 								)= explode('|',$blc_value);
2026: 							if ($d['licenses'][$key]['b_l_c'][$blc_key]['tariff'] == chr(2)) 
2027: 							{
2028: 								$d['licenses'][$key]['b_l_c'][$blc_key]['tariff'] = NULL;
2029: 							}
2030: 							if ($d['licenses'][$key]['b_l_c'][$blc_key]['currency'] == chr(2)) 
2031: 							{
2032: 								$d['licenses'][$key]['b_l_c'][$blc_key]['currency'] = NULL;
2033: 							}
2034: 						}
2035: 					}
2036: 				}
2037: 				$out['car'][$d['c_id']] = $d;
2038: 			}
2039: 			if (empty($this->associativeArray)) $out['car'] =  array_values($out['car']);
2040: 
2041: 			return array(
2042: 				'code' 		=>	'200',
2043: 				'status' 	=>	'success',		
2044: 				'data' 		=>	$out,
2045: 				'auth_user' =>	array(
2046: 									'u_id' => $_SESSION[UID],
2047: 									'u_name' => $_SESSION['name'],
2048: 									'u_family' => $_SESSION['family'],
2049: 									'u_middle' => $_SESSION['middle'],
2050: 									'u_email' => $_SESSION['email'],
2051: 									'u_phone' => $_SESSION['phone'],
2052: 									'u_role' => $_SESSION['id_role'],
2053: 									'u_a_role' => $this->id_role,
2054: 									'u_check_state' => $_SESSION['id_verification_status'],
2055: 									'u_ban' => $_SESSION['user_ban_status'],
2056: 									'u_active' => $_SESSION['active'],
2057: 									'u_photo' => $_SESSION['photo_link'],
2058: 									'u_birthday' => $_SESSION['birthday_date'],
2059: 									'u_lang' => $_SESSION['id_lang'],
2060: 									'u_currency' => $_SESSION['currency'],
2061: 									'u_gps_software' => $_SESSION['id_navigation']
2062: 								)
2063: 			);
2064: 		}
```

### `selectCart` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `17441-17557`
```php
17441: 		public function selectCart($filter = NULL)
17442: 		{
17443: 			if (empty($_SESSION[UID])) {
17444: 				return $this->showError('404', 'error', 'unauthorized access');
17445: 			}
17446: 			$sql_user = "`id_user` as u_id,";
17447: 			if ($filter == 'all')
17448: 			{
17449: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
17450: 				$sql_where = "1 = 1";
17451: 			}
17452: 			elseif ($filter == 'trip')	
17453: 			{
17454: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter');
17455: 				$sql_where = "`trip`.`driver` = '" . $_SESSION[UID] . "'";
17456: 			}
17457: 			else
17458: 			{
17459: 				$sql_where = "`id_user` = '" . $_SESSION[UID] . "'";
17460: 				$sql_user = "";
17461: 			}
17462: 
17463: 			$s = "SELECT 
17464: 					$sql_user
17465: 					`product` as prod,
17466: 					`ticket`.`id_seat` as prop,
17467: 					`count`,
17468: 					`booking_limit`,
17469: 					`trip`.`from` as sc_id,
17470: 					`trip`.`json`,
17471: 					IF(`order`.`id_order` IS NULL AND `ticket`.`status` NOT in (2,3),IF(`ticket`.`tariff` IS NULL,NULL,IF(`ticket`.`currency` IS NULL,`ticket`.`tariff`,CONCAT_WS(' ',`ticket`.`tariff`,`ticket`.`currency`))),'taken') as price,
17472: 					`complex_update`
17473: 				FROM `cart`
17474: 				LEFT JOIN `trip` ON `trip`.`id_trip` = `cart`.`product`
17475: 				LEFT JOIN `ticket` ON `ticket`.`id_trip` = `cart`.`product` AND `ticket`.`id_trip_seat` = `cart`.`property`
17476: 				LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`
17477: 				WHERE
17478: 					$sql_where
17479: 				";		
17480: 	
17481: 			$q = query($s);
17482: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
17483: 			
17484: 			$out = array('cart' => array());
17485: 			while ($d = fetch_assoc($q))
17486: 			{
17487: 				$sc_id = explode(chr(0),$d['sc_id']);
17488: 				if (count($sc_id) > 1 || $sc_id[0] == 'sc_id')
17489: 				{
17490: 					$d['sc_id'] = $sc_id[1];
17491: 				}
17492: 				else
17493: 				{
17494: 					unset($d['sc_id']);
17495: 				}
17496: 				if (isset($d['price']))
17497: 				{		
17498: 					if ($d['price'] == 'taken') $d['busy'] = 1;
17499: 				}
17500: 				else
17501: 				{
17502: 					$d['json'] = json_decode($d['json'],true);
17503: 					if (!empty($d['json']['seats_sold']))
17504: 					{
17505: 						foreach($d['json']['seats_sold'] as $seat=>$val)
17506: 						{
17507: 							if ($seat == $d['prop'])
17508: 							{
17509: 								if (is_array($val))
17510: 								{
17511: 									if (count($val) > 0 && isset($d['json']['price'][$val[0]]))
17512: 									{
17513: 										$d['price'] = $d['json']['price'][$val[0]];
17514: 									}
17515: 									if (count($val) > 1) $d['busy'] = 1;
17516: 								}
17517: 								else
17518: 								{
17519: 									if (isset($d['json']['price'][$val]))
17520: 									{
17521: 										$d['price'] = $d['json']['price'][$val];
17522: 									}
17523: 								}
17524: 							}
17525: 						}
17526: 					}
17527: 				}
17528: 				unset($d['json']);
17529: 				$d['count'] = (int)$d['count'];
17530: 				add_time_zone($d['booking_limit']);
17531: 				$out['cart'][] = $d;
17532: 			}
17533: 
17534: 			return array(
17535: 				'code' 		=>	'200',
17536: 				'status' 	=>	'success',		
17537: 				'data' 		=>	$out,
17538: 				'auth_user' =>	array(
17539: 									'u_id' => $_SESSION[UID],
17540: 									'u_name' => $_SESSION['name'],
17541: 									'u_family' => $_SESSION['family'],
17542: 									'u_middle' => $_SESSION['middle'],
17543: 									'u_email' => $_SESSION['email'],
17544: 									'u_phone' => $_SESSION['phone'],
17545: 									'u_role' => $_SESSION['id_role'],
17546: 									'u_a_role' => $this->id_role,
17547: 									'u_check_state' => $_SESSION['id_verification_status'],
17548: 									'u_ban' => $_SESSION['user_ban_status'],
17549: 									'u_active' => $_SESSION['active'],
17550: 									'u_photo' => $_SESSION['photo_link'],
17551: 									'u_birthday' => $_SESSION['birthday_date'],
17552: 									'u_lang' => $_SESSION['id_lang'],
17553: 									'u_currency' => $_SESSION['currency'],
17554: 									'u_gps_software' => $_SESSION['id_navigation']
17555: 								)
17556: 			);
17557: 		}
```

### `selectCartBlock` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `18610-18750`
```php
18610: 		public function selectCartBlock($filter = NULL)
18611: 		{
18612: 			if (empty($_SESSION[UID])) {
18613: 				return $this->showError('404', 'error', 'unauthorized access');
18614: 			}
18615: 
18616: 			$sql_group_concat = "CONCAT_WS(0x00,
18617: 										`cart_block_trip`.`id_trip`,
18618: 										IFNULL(`cart_block_trip`.`status`,0x02),
18619: 										`cart_block_trip`.`status_sys`
18620: 									)";
18621: 			 $sql_left_trip = $sql_left = "";
18622: 			if ($filter == 'all')
18623: 			{
18624: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter all');
18625: 				$sql_field = "`cart_block`.`id_user` as u_id,
18626: 				`cart_block`.`active`,";
18627: 				if ($this->id_role != 4)
18628: 				{
18629: 					$sql_group_concat = "IF(`trip`.`driver` = '{$_SESSION[UID]}',
18630: 								CONCAT_WS(0x00,
18631: 									`cart_block_trip`.`id_trip`,
18632: 									IFNULL(`cart_block_trip`.`status`,0x02),
18633: 									`cart_block_trip`.`status_sys`
18634: 								),NULL)";
18635: 				}
18636: 				$sql_left_trip = "LEFT JOIN `trip` USING (`id_trip`)";
18637: 				$sql_where = "1 = 1";
18638: 			}
18639: 			elseif ($filter == 'trip')
18640: 			{
18641: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter trip');
18642: 				$sql_field = "`cart_block`.`id_user` as u_id,
18643: 				`cart_block`.`active`,";
18644: 				if ($this->id_role != 4)
18645: 				{
18646: 					$sql_group_concat = "IF(`trip`.`driver` = '{$_SESSION[UID]}',
18647: 								CONCAT_WS(0x00,
18648: 									`cart_block_trip`.`id_trip`,
18649: 									IFNULL(`cart_block_trip`.`status`,0x02),
18650: 									`cart_block_trip`.`status_sys`
18651: 								),NULL)";
18652: 				}
18653: 				$sql_left_trip = "LEFT JOIN `trip` USING (`id_trip`)";
18654: 				$sql_left = "LEFT JOIN (
18655: 				SELECT
18656: 					`schedule`.`id_schedule`
18657: 				FROM `schedule`
18658: 				LEFT JOIN `trip` ON `trip`.`id_schedule` = `schedule`.`id_schedule`
18659: 				WHERE
18660: 					`schedule`.`active` = '1' AND `trip`.`driver` = '{$_SESSION[UID]}'
18661: 				GROUP BY
18662: 					`schedule`.`id_schedule`
18663: 				ORDER BY `schedule`.`id_schedule` DESC
18664: 				) sc ON sc.`id_schedule` = `cart_block`.`product`";
18665: 				$sql_where = "sc.`id_schedule` IS NOT NULL";
18666: 			}
18667: 			else
18668: 			{
18669: 				$sql_field = "";
18670: 				$sql_where = "`cart_block`.`id_user` = '" . $_SESSION[UID] . "' AND `cart_block`.`active` = 1";
18671: 			}
18672: 
18673: 			$s = "SELECT 
18674: 					$sql_field
18675: 					`cart_block`.`product` as prod,
18676: 					`cart_block`.`property` as prop,
18677: 					`cart_block`.`count`, 
18678: 					`cart_block`.`create_datetime` as created,
18679: 					`cart_block`.`notification` as notice,
18680: 					`cart_block`.`status`,
18681: 					`cart_block`.`status_sys`,
18682: 					(SELECT
18683: 						IF(`cart_block_trip`.`id_cart_block` IS NULL,NULL,
18684: 							GROUP_CONCAT(
18685: 								$sql_group_concat 
18686: 							SEPARATOR 0x01) 
18687: 						)
18688: 					 FROM `cart_block_trip`
18689: 					 $sql_left_trip
18690: 					 WHERE `cart_block_trip`.`id_cart_block` = `cart_block`.`id_cart_block`
18691: 					) as statuses
18692: 				FROM `cart_block`
18693: 				$sql_left
18694: 				WHERE
18695: 					$sql_where
18696: 				";
18697: 
18698: 			$q = query($s);
18699: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
18700: 			
18701: 			$out = array('cart' => array());
18702: 			while ($d = fetch_assoc($q))
18703: 			{
18704: 				$d['count'] = (int)$d['count'];
18705: 				add_time_zone($d['created']);	
18706: 				if (!empty($d['statuses']))
18707: 				{
18708: 					$d['statuses'] = explode(chr(1),$d['statuses']);
18709: 					foreach ($d['statuses'] as $key=>$value)
18710: 					{
18711: 						$d['statuses'][$key] = array();
18712: 						list(
18713: 							$d['statuses'][$key]['t_id'],
18714: 							$d['statuses'][$key]['status'],
18715: 							$d['statuses'][$key]['status_sys'],
18716: 							)= explode(chr(0),$value);
18717: 
18718: 						if ($d['statuses'][$key]['status'] === chr(2))
18719: 						{
18720: 							$d['statuses'][$key]['status'] = NULL;
18721: 						}
18722: 					}
18723: 				}
18724: 				$out['cart'][] = $d;
18725: 			}
18726: 
18727: 			return array(
18728: 				'code' 		=>	'200',
18729: 				'status' 	=>	'success',		
18730: 				'data' 		=>	$out,
18731: 				'auth_user' =>	array(
18732: 									'u_id' => $_SESSION[UID],
18733: 									'u_name' => $_SESSION['name'],
18734: 									'u_family' => $_SESSION['family'],
18735: 									'u_middle' => $_SESSION['middle'],
18736: 									'u_email' => $_SESSION['email'],
18737: 									'u_phone' => $_SESSION['phone'],
18738: 									'u_role' => $_SESSION['id_role'],
18739: 									'u_a_role' => $this->id_role,
18740: 									'u_check_state' => $_SESSION['id_verification_status'],
18741: 									'u_ban' => $_SESSION['user_ban_status'],
18742: 									'u_active' => $_SESSION['active'],
18743: 									'u_photo' => $_SESSION['photo_link'],
18744: 									'u_birthday' => $_SESSION['birthday_date'],
18745: 									'u_lang' => $_SESSION['id_lang'],
18746: 									'u_currency' => $_SESSION['currency'],
18747: 									'u_gps_software' => $_SESSION['id_navigation']
18748: 								)
18749: 			);
18750: 		}
```

### `selectCheckTicketLog` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `24450-24528`
```php
24450: 		public function selectCheckTicketLog($id_schedule = '', $seat = '', $id_user = '', $pass = NULL)
24451: 		{
24452: 			if (empty($_SESSION[UID])) {
24453: 				return $this->showError('404', 'error', 'unauthorized access');
24454: 			}
24455: 
24456: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24457: 			
24458: 			$sql_where = array();
24459: 			if (!empty($id_schedule)) 
24460: 			{
24461: 				$sql_where[] =  "`ticket`.`id_schedule` in ($id_schedule)";
24462: 			}
24463: 			if (!empty($seat)) 
24464: 			{
24465: 				$seat = json_decode($seat,true);
24466: 				
24467: 				if (empty($seat) || !is_array($seat)) 
24468: 				{
24469: 					return $this->showError('404', 'error', 'wrong seat');
24470: 				}
24471: 
24472: 				$sql_seat = array();
24473: 				foreach($seat as $s_el)
24474: 				{
24475: 					$sql_seat[] = "'$s_el'";
24476: 				}
24477: 				$sql_seat = implode(',',$sql_seat);
24478: 				$sql_where[] = "`ticket`.`id_seat` in ($sql_seat)";
24479: 			}
24480: 			if (!empty($id_user)) 
24481: 			{
24482: 				$sql_where[] =  "`ticket_pass_log`.`id_user` in ($id_user)";
24483: 			}
24484: 			if (isset($pass)) 
24485: 			{
24486: 				$sql_where[] =  "`ticket_pass_log`.`pass` in ($pass)";
24487: 			}
24488: 
24489: 			if (empty($sql_where))
24490: 			{
24491: 				$sql_where = 'WHERE 
24492: 					 `ticket_pass_log`.`id_trip` IS NOT NULL';
24493: 			}
24494: 			else
24495: 			{
24496: 				$sql_where = "WHERE 
24497: 					 `ticket_pass_log`.`id_trip` IS NOT NULL AND " . implode(' AND ',$sql_where);
24498: 			}
24499: 
24500: 			$s = "SELECT
24501: 					`ticket`.`id_schedule` as sc_id,
24502: 					`ticket`.`id_seat` as seat,
24503: 					`ticket`.`id_trip` as t_id,
24504: 					`ticket`.`id_trip_seat` as ti_t_id,
24505: 					`ticket_pass_log`.`id_user` as u_id,
24506: 					`ticket_pass_log`.`pass`,
24507: 					`ticket_pass_log`.`int_timestamp` as datetime
24508: 				FROM `ticket`
24509: 				LEFT JOIN `ticket_pass_log` ON `ticket_pass_log`.`id_trip` = `ticket`.`id_trip` AND `ticket_pass_log`.`id_trip_seat` = `ticket`.`id_trip_seat`
24510: 				$sql_where
24511: 				";
24512: 
24513: 				$q = query($s);
24514: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
24515: 
24516: 				$out = array('log' => array());
24517: 				while ($d = fetch_assoc($q))
24518: 				{
24519: 					$d['datetime'] = (new DateTime("@{$d['datetime']}"))->setTimezone(new DateTimeZone(MYSQL_TIME_ZONE))->format('Y_m_d-H_i_sP');
24520: 					$out['log'][] = $d;
24521: 				}
24522: 
24523: 			return array(
24524: 				'code' 		=>	'200',
24525: 				'status' 	=>	'success',		
24526: 				'data' 		=>	$out
24527: 			);
24528: 		}
```

### `selectContact` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20828-21105`
```php
20828: 		public function selectContact($id_contact_item = NULL, $id_user = NULL, $id_owner_type = NULL, $id_contact_type = NULL, $owner_types = array(), $langs = array())
20829: 		{	
20830: 			if (empty($_SESSION[UID])) {
20831: 				return $this->showError('404', 'error', 'unauthorized access');
20832: 			}
20833: 
20834: 			$sql_list = array();
20835: 			$sql_where = array();
20836: 			if (!empty($id_contact_type))
20837: 			{
20838: 				$sql_where[] = "`contact_items`.`id_contact_type` in ($id_contact_type)";
20839: 			}
20840: 			if (empty($id_owner_type))
20841: 			{
20842: 				$sql_where_owner_type = '';
20843: 				$id_owner_type = array_keys($owner_types);
20844: 			}
20845: 			else
20846: 			{
20847: 				$sql_where_owner_type = "`contact_items`.`id_owner_type` in ($id_owner_type)";
20848: 				$id_owner_type = explode(',',$id_owner_type);
20849: 			}
20850: 			if (!empty($id_contact_item))
20851: 			{
20852: 				$sql_where[] = "`contact_items`.`id_contact_item` in ($id_contact_item)";
20853: 			}
20854: 			$sql_private_o_type_all = $sql_private_o_type_1 = $sql_private_o_type_other = ",`contact_items`.`contact_number` as 'number',
20855: 						`contact_items`.`key1`,
20856: 						`contact_items`.`key2`,
20857: 						`contact_items`.`token`,
20858: 						`contact_items`.`hash`,
20859: 						`contact_items`.`secret`,
20860: 						`contact_items`.`host`,
20861: 						`contact_items`.`port`,
20862: 						`contact_items`.`login`,
20863: 						`contact_items`.`password`,
20864: 						`contact_items`.`smtpsecure`,
20865: 						`contact_items`.`fromname`";
20866: 			if ($this->id_role != 4) $sql_private_o_type_other = ",NULL as 'number',
20867: 						NULL as 'key1',
20868: 						NULL as 'key2',
20869: 						NULL as 'token',
20870: 						NULL as 'hash',
20871: 						NULL as 'secret',
20872: 						NULL as 'host',
20873: 						NULL as 'port',
20874: 						NULL as 'login',
20875: 						NULL as 'password',
20876: 						NULL as 'smtpsecure',
20877: 						NULL as 'fromname'";
20878: 			if ($id_user === NULL)
20879: 			{
20880: 				if ($this->id_role != 4) $id_user = $_SESSION[UID];
20881: 			}
20882: 			else
20883: 			{
20884: 				if ($this->id_role != 4 && $id_user != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
20885: 			}
20886: 			$sql_name = $sql_description = array();
20887: 			foreach ($langs as $lang)
20888: 			{
20889: 				$sql_name[] = "`contact_items`.`name_{$lang['iso']}` as {$lang['iso']}";
20890: 				$sql_description[] = "`contact_items`.`description_{$lang['iso']}` as about_{$lang['iso']}";
20891: 			}
20892: 			$sql_name = implode(',',$sql_name);
20893: 			$sql_description = implode(',',$sql_description);
20894: 			if ($id_user === NULL)
20895: 			{
20896: 				$sql_list[] = array(
20897: 								'field' => $sql_private_o_type_all,
20898: 								'left_join' => '',
20899: 								'where' => $sql_where,
20900: 								'where_owner_type' => $sql_where_owner_type,
20901: 								'group' => ''
20902: 				);
20903: 			}
20904: 			else
20905: 			{
20906: 				$sql_list = array();
20907: 				foreach($id_owner_type as $o_type)
20908: 				{
20909: 					if (empty($owner_types[$o_type])) return $this->showError('404', 'error', "owner type $o_type not found");
20910: 					$table = $owner_types[$o_type]['table'];
20911: 					$column = $owner_types[$o_type]['column'];
20912: 					$sql = $owner_types[$o_type]['sql'];
20913: 					if ($table !== NULL && $table !== "" && $column!== NULL && $column!== "")
20914: 					{
20915: 						if ($table == 'users')
20916: 						{
20917: 							if ($column == 'id_user')
20918: 							{
20919: 								$id_user_str = str_replace(',',"','",$id_user);
20920: 								$id_user_str = "'$id_user_str'";
20921: 								$sql_list[] = array(
20922: 												'field' => "$sql_private_o_type_1,`owner` as u_id",
20923: 												'left_join' => '',
20924: 												'where' => $sql_where,
20925: 												'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND `contact_items`.`owner` in ($id_user_str))",
20926: 												'group' => ''
20927: 								);
20928: 								
20929: 							}
20930: 							elseif ($column == 'id_role')
20931: 							{
20932: 								if ($id_user == $_SESSION[UID])
20933: 								{
20934: 									$id_role = $this->id_role;
20935: 									$sql_list[] = array(
20936: 													'field' => "$sql_private_o_type_other,`owner` as u_id",
20937: 													'left_join' => '',
20938: 													'where' => $sql_where,
20939: 													'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND `contact_items`.`owner` = '$id_role')",
20940: 													'group' => ''
20941: 									);
20942: 								}
20943: 								else
20944: 								{							
20945: 									$sql_list[] = array(
20946: 													'field' => "$sql_private_o_type_other,GROUP_CONCAT(ow.`id_user` SEPARATOR ',') as u_id",
20947: 													'left_join' => "LEFT JOIN (
20948: 																SELECT
20949: 																	`id_user`,
20950: 																	CONCAT(`id_role`,'') as owner
20951: 																FROM
20952: 																	`users`
20953: 																WHERE
20954: 																	`id_user` in ($id_user)
20955: 															) ow USING (`owner`)",
20956: 													'where' => $sql_where,
20957: 													'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND ow.`id_user` in ($id_user))",
20958: 													'group' => "GROUP BY `contact_items`.`id_contact_item`"
20959: 									);
20960: 								}
20961: 							}
20962: 							elseif (array_key_exists($column,$_SESSION))
20963: 							{
20964: 								$sql_list[] = array(
20965: 												'field' => "$sql_private_o_type_other,GROUP_CONCAT(ow.`id_user` SEPARATOR ',') as u_id",
20966: 												'left_join' => "LEFT JOIN (
20967: 															SELECT
20968: 																`id_user`,
20969: 																CONCAT(`$column`,'') as owner
20970: 															FROM
20971: 																`users`
20972: 															WHERE
20973: 																`id_user` in ($id_user)
20974: 														) ow USING (`owner`)",
20975: 												'where' => $sql_where,
20976: 												'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND ow.`id_user` in ($id_user))",
20977: 												'group' => "GROUP BY `contact_items`.`id_contact_item`"
20978: 								);
20979: 							}
20980: 							else
20981: 							{
20982: 								$sql_list[] = array(
20983: 												'field' => "$sql_private_o_type_other,GROUP_CONCAT(ow.`id_user` SEPARATOR ',') as u_id",
20984: 												'left_join' => "LEFT JOIN (
20985: 															SELECT
20986: 																`id_user`,
20987: 																CONCAT(`$column`,'') as owner
20988: 															FROM
20989: 																`users`
20990: 															WHERE
20991: 																`id_user` in ($id_user)
20992: 														) ow USING (`owner`)",
20993: 												'where' => $sql_where,
20994: 												'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND ow.`id_user` in ($id_user))",
20995: 												'group' => "GROUP BY `contact_items`.`id_contact_item`"
20996: 								);
20997: 							}
20998: 						}
20999: 						else
21000: 						{
21001: 
21002: 							$delimiter = $owner_types[$o_type]['delimiter'] === NULL ? ',' : $owner_types[$o_type]['delimiter'];
21003: 							$delimiter = real_escape_string($delimiter);
21004: 							$sql_list[] = array(
21005: 											'field' => "$sql_private_o_type_other,GROUP_CONCAT(ow.`id_user` SEPARATOR ',') as u_id",
21006: 											'left_join' => "LEFT JOIN (
21007: 														SELECT
21008: 															`id_user`,
21009: 															CONCAT(GROUP_CONCAT(`$column` SEPARATOR '$delimiter'),'') as owner
21010: 														FROM
21011: 															`$table`
21012: 														WHERE
21013: 															`id_user` in ($id_user)
21014: 														GROUP BY
21015: 															`id_user`
21016: 														ORDER BY
21017: 															`$column`
21018: 													) ow USING (`owner`)",
21019: 											'where' => $sql_where,
21020: 											'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND ow.`id_user` in ($id_user))",
21021: 											'group' => "GROUP BY `contact_items`.`id_contact_item`"
21022: 							);
21023: 						}
21024: 					}
21025: 					else
21026: 					{		
21027: 						if ($sql === NULL) continue;
21028: 						$sql = str_replace('{$id_user}',$id_user,$sql);
21029: 						$sql_list[] = array(
21030: 										'field' => "$sql_private_o_type_other,GROUP_CONCAT(ow.`id_user` SEPARATOR ',') as u_id",
21031: 										'left_join' => "LEFT JOIN ($sql) ow USING (`owner`)",
21032: 										'where' => $sql_where,
21033: 										'where_owner_type' => "(`contact_items`.`id_owner_type` = $o_type AND ow.`id_user` in ($id_user))",
21034: 										'group' => "GROUP BY `contact_items`.`id_contact_item`"
21035: 						);
21036: 					}
21037: 				}
21038: 			}
21039: 			if (empty($sql_list)) return $this->showError('404', 'error', "valid owner type not found");
21040: 			$s = array();
21041: 			foreach($sql_list as $list_val)
21042: 			{
21043: 				$sql_field = $list_val['field'];
21044: 				$sql_left_join = $list_val['left_join'];
21045: 				$sql_where = $list_val['where'];
21046: 				$sql_where[] = $list_val['where_owner_type'];
21047: 				$sql_where = implode(' AND ',$sql_where);
21048: 				if (!empty($sql_where)) $sql_where = "WHERE
```

### `selectDataPrivate` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `17398-17439`
```php
17398: 		public function selectDataPrivate($data_private = array(), $json_like = '')
17399: 		{
17400: 			if (empty($_SESSION[UID])) {
17401: 				return $this->showError('404', 'error', 'unauthorized access');
17402: 			}
17403: 			if ($this->id_role != 4)
17404: 			{
17405: 				return $this->showError('404', 'error', 'not enough rights');
17406: 			}
17407: 
17408: 			if (empty($json_like))
17409: 			{
17410: 				$data_private_out = $data_private;
17411: 			}
17412: 			else
17413: 			{
17414: 				@$json_like = json_decode($json_like,true);
17415: 				if (empty($json_like) || !is_array($json_like)) 
17416: 				{
17417: 					return $this->showError('404', 'error', 'wrong json_like');
17418: 				}
17419: 				find_arr_like($data_private,$json_like);
17420: 				$data_private_out = $json_like;
17421: 			}
17422: 			
17423: 			if (isset($data_private_out['script_templates']))
17424: 			{
17425: 				foreach($data_private_out['script_templates'] as $id=>$template)
17426: 				{
17427: 					if (!empty($template['file']))
17428: 					{
17429: 						$data_private_out['script_templates'][$id]['file'] = file_get_contents($_SERVER['DOCUMENT_ROOT'] . $template['file']);
17430: 					}
17431: 				}
17432: 			}
17433: 
17434: 			return array(
17435: 				'code' 		=>	'200',
17436: 				'status' 	=>	'success',		
17437: 				'data' 		=>	$data_private_out
17438: 			);
17439: 		}
```

### `selectDropboxFile` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `13991-14069`
```php
13991: 		public function selectDropboxFile($id_dropbox_link = '')
13992: 		{
13993: 			if (empty($id_dropbox_link)) 
13994: 			{
13995: 				return $this->showError('404', 'error', 'empty dl_id');
13996: 			}
13997: 			if (empty($_SESSION[UID]))
13998: 			{
13999: 				$s = "SELECT
14000: 						`id_dropbox_link`,
14001: 						`json`,
14002: 						`private`,
14003: 						`deleted`
14004: 					FROM `dropbox_link` 				
14005: 					WHERE	
14006: 						`id_dropbox_link` = '" . $id_dropbox_link  . "'
14007: 					LIMIT 1
14008: 					";
14009: 			}
14010: 			elseif ($this->id_role == 4)
14011: 			{
14012: 				$s = "SELECT
14013: 						`id_dropbox_link`,
14014: 						`json`,
14015: 						`deleted`
14016: 					FROM `dropbox_link` 				
14017: 					WHERE	
14018: 						`id_dropbox_link` = '" . $id_dropbox_link  . "'
14019: 					LIMIT 1
14020: 					";
14021: 			}
14022: 			else
14023: 			{
14024: 				$s = "SELECT
14025: 						`id_dropbox_link`,
14026: 						`json`,
14027: 						`private`,
14028: 						`deleted`,
14029: 						IF(`deleted` = 0 AND `private` = '1',(
14030: 							SELECT
14031: 								`id_user`
14032: 							FROM
14033: 								`users_dropbox_link`
14034: 							WHERE
14035: 								`users_dropbox_link`.`id_dropbox_link` = `dropbox_link`.`id_dropbox_link` AND 
14036: 								`id_user` = '" . $_SESSION[UID] . "'
14037: 						),'') as user
14038: 					FROM `dropbox_link` 				
14039: 					WHERE	
14040: 						`id_dropbox_link` = '" . $id_dropbox_link  . "'
14041: 					LIMIT 1
14042: 					";
14043: 			}
14044: 
14045: 			$q = query($s);
14046: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
14047: 
14048: 			$d = fetch_assoc($q);
14049: 			if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link not found');
14050: 			if (!empty($d['deleted']))return $this->showError('404', 'error', 'deleted dropbox link');
14051: 			if (empty($_SESSION[UID]))
14052: 			{
14053: 				if ($d['private'] != -1)
14054: 				{
14055: 					return $this->showError('404', 'error', 'unauthorized access');
14056: 				}
14057: 			}
14058: 			else
14059: 			{
14060: 				if (isset($d['private']) && $d['private'] == 1 && empty($d['user']))
14061: 				{
14062: 					return $this->showError('404', 'error', 'not enough rights');
14063: 				}
14064: 			}
14065: 
14066: 			$json = json_decode($d['json'],true);
14067: 			$response = download_from_dropbox($json['name_upload'],$id_dropbox_link,$json['type']);
14068: 			if (!empty($response['error'])) return $this->showError('404', 'error', $response['error']);
14069: 		}
```

### `selectFavoriteUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `10641-10790`
```php
10641: 		public function selectFavoriteUser($id_user = "")
10642: 		{
10643: 			if (empty($_SESSION[UID])) {
10644: 				return $this->showError('404', 'error', 'unauthorized access');
10645: 			}
10646: 			$sql_add = "
10647: 					`users`.`phone` as u_phone,
10648: 					`users`.`phone_is_verified` as u_phone_checked,
10649: 					`users`.`email` as u_email,
10650: 					`users`.`birthday_date` as u_birthday,
10651: 					`users`.`referral_code` as ref_code,
10652: 					`users`.`referrer` as referrer_u_id,
10653: 					`tg` as u_tg,
10654: 					`tg_is_verified` as u_tg_checked,
10655: 					`id_user_upper` as u_upper,
10656: 					`wa` as u_wa,
10657: 					`wa_is_verified` as u_wa_checked,";
10658: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10659: 			
10660: 			{
10661: 				$id_user = $_SESSION[UID];
10662: 				if ($this->id_role != 4) $sql_add = "";
10663: 			}
10664: 			else
10665: 			{	
10666: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10667: 			}
10668: 
10669: 			$s = "SELECT 
10670: 					`users`.`id_user` as u_id,
10671: 					`users`.`id_role` as u_role,
10672: 					`users`.`name` as u_name,
10673: 					`users`.`family` as u_family,
10674: 					`users`.`middle` as u_middle,
10675: 					" . $sql_add . "
10676: 					`users`.`photo_link` as u_photo,
10677: 					`users`.`id_lang` as u_lang,
10678: 					`users`.`currency` as u_currency,
10679: 					`users`.`id_city` as u_city,
10680: 					`users`.`tips` as u_tips,
10681: 					`users`.`language_skills` as u_lang_skills,
10682: 					`users`.`description` as u_description,
10683: 					`users`.`id_navigation` as u_gps_software,
10684: 					`users`.`id_verification_status` as u_check_state,
10685: 					`users`.`active` as u_active,			
10686: 					`users`.`out_order` as out_drive,
10687: 					`users`.`out_order_to` as out_address,
10688: 					`users`.`out_order_to_lat` as out_latitude,
10689: 					`users`.`out_order_to_lng` as out_longitude,
10690: 					`users`.`out_order_complete_datetime` as out_est_datetime,
10691: 					`users`.`out_order_from` as out_s_address,
10692: 					`users`.`out_order_from_lat` as out_s_latitude,
10693: 					`users`.`out_order_from_lng` as out_s_longitude,
10694: 					`users`.`out_order_passengers_count` as out_passengers,
10695: 					`users`.`out_order_luggage_count` as out_luggage,
10696: 					`json` as u_details,
10697: 					(SELECT
10698: 						GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
10699: 					 FROM `users_order_comment`
10700: 					 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
10701: 					) as b_comments,
10702: 					(SELECT
10703: 						GROUP_CONCAT(`users_service`.`id_service` SEPARATOR ',')
10704: 					 FROM `users_service`
10705: 					 WHERE `users_service`.`id_user` = `users`.`id_user`
10706: 					) as b_services,
10707: 					(SELECT
10708: 						GROUP_CONCAT(
10709: 							CONCAT_WS('|',`users_order_location`.`id_order_location`,`users_order_location`.`basic`) 
10710: 						SEPARATOR ';')
10711: 					 FROM `users_order_location`
10712: 					 WHERE `users_order_location`.`id_user` = `users`.`id_user`
10713: 					) as b_location_classes,
10714: 					`id_schedule` as sc_id
10715: 				FROM `users`
10716: 				LEFT JOIN `users_favorite` ON `users_favorite`.`id_favorite` = `users`.`id_user`
10717: 				WHERE
10718: 					`users`.`deleted` = '0' AND `users_favorite`.`id_user` = '" . $id_user . "'
10719: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
10720: 				";
10721: 
10722: 			$q = query($s);
10723: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
10724: 			
10725: 			$out = array('user' => array());
10726: 
10727: 			while ($d = fetch_assoc($q))
10728: 			{
10729: 				$d['u_ban'] = get_user_ban_status($d['u_id']);
10730: 
10731: 				$d['u_photo'] = $d['u_photo'] ? url($d['u_photo'],FILE_PATH) : '';
10732: 
10733: 				add_time_zone($d['out_est_datetime']);
10734: 
10735: 				if (!empty($d['b_comments'])) $d['b_comments'] = explode(',',$d['b_comments']);		
10736: 
10737: 				if (!empty($d['b_services'])) $d['b_services'] = explode(',',$d['b_services']);	
10738: 				
10739: 				if (!empty($d['b_location_classes']))
10740: 				{
10741: 					$d['b_location_classes'] = explode(';',$d['b_location_classes']);
10742: 					foreach ($d['b_location_classes'] as $key=>$value)
10743: 					{
10744: 						$d['b_location_classes'][$key] = array();
10745: 						list(
10746: 							$d['b_location_classes'][$key]['b_location_class'],
10747: 							$d['b_location_classes'][$key]['basic']
10748: 							)= explode('|',$value);
10749: 						$d['b_location_classes'][$key]['basic'] = (int)$d['b_location_classes'][$key]['basic'];
10750: 					}
10751: 				}
10752: 
10753: 				if (isset($d['u_phone_checked'])) $d['u_phone_checked'] = (int)$d['u_phone_checked'];
10754: 
10755: 				$d['u_active'] = (int)$d['u_active'];		
10756: 				
10757: 				$d['out_drive'] = (int)$d['out_drive'];	
10758: 
10759: 				$d['u_details'] = json_decode($d['u_details'],true);
10760: 
10761: 
10762: 				$out['user'][$d['u_id']] = $d;
10763: 			}
10764: 
10765: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
10766: 
10767: 			return array(
10768: 				'code' 		=>	'200',
10769: 				'status' 	=>	'success',		
10770: 				'data' 		=>	$out,
10771: 				'auth_user' =>	array(
10772: 									'u_id' => $_SESSION[UID],
10773: 									'u_name' => $_SESSION['name'],
10774: 									'u_family' => $_SESSION['family'],
10775: 									'u_middle' => $_SESSION['middle'],
10776: 									'u_email' => $_SESSION['email'],
10777: 									'u_phone' => $_SESSION['phone'],
10778: 									'u_role' => $_SESSION['id_role'],
10779: 									'u_a_role' => $this->id_role,
10780: 									'u_check_state' => $_SESSION['id_verification_status'],
10781: 									'u_ban' => $_SESSION['user_ban_status'],
10782: 									'u_active' => $_SESSION['active'],
10783: 									'u_photo' => $_SESSION['photo_link'],
10784: 									'u_birthday' => $_SESSION['birthday_date'],
10785: 									'u_lang' => $_SESSION['id_lang'],
10786: 									'u_currency' => $_SESSION['currency'],
10787: 									'u_gps_software' => $_SESSION['id_navigation']
10788: 								)
10789: 			);
10790: 		}
```

### `selectInnerUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20503-20636`
```php
20503: 		public function selectInnerUser($id_user = "")
20504: 		{
20505: 			if (empty($_SESSION[UID])) {
20506: 				return $this->showError('404', 'error', 'unauthorized access');
20507: 			}
20508: 			$sql_add = "
20509: 					`phone` as u_phone,
20510: 					`phone_is_verified` as u_phone_checked,
20511: 					`email` as u_email,
20512: 					`birthday_date` as u_birthday,
20513: 					`referral_code` as ref_code,
20514: 					`referrer` as referrer_u_id,
20515: 					`tg` as u_tg,
20516: 					`tg_is_verified` as u_tg_checked,
20517: 					`id_user_upper` as u_upper,
20518: 					`wa` as u_wa,
20519: 					`wa_is_verified` as u_wa_checked,";
20520: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
20521: 			{
20522: 				$id_user = $_SESSION[UID];
20523: 				if ($this->id_role != 4) $sql_add = "";
20524: 			}
20525: 			else
20526: 			{	
20527: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
20528: 			}
20529: 
20530: 			$s = "SELECT 
20531: 					`id_user` as u_id,
20532: 					`id_role` as u_role,
20533: 					`name` as u_name,
20534: 					`family` as u_family,
20535: 					`middle` as u_middle,
20536: 					" . $sql_add . "
20537: 					`photo_link` as u_photo,
20538: 					`id_lang` as u_lang,
20539: 					`currency` as u_currency,
20540: 					`id_city` as u_city,
20541: 					`tips` as u_tips,
20542: 					`language_skills` as u_lang_skills,
20543: 					`description` as u_description,
20544: 					`id_navigation` as u_gps_software,
20545: 					`id_verification_status` as u_check_state,
20546: 					`active` as u_active,			
20547: 					`out_order` as out_drive,
20548: 					`out_order_to` as out_address,
20549: 					`out_order_to_lat` as out_latitude,
20550: 					`out_order_to_lng` as out_longitude,
20551: 					`out_order_complete_datetime` as out_est_datetime,
20552: 					`out_order_from` as out_s_address,
20553: 					`out_order_from_lat` as out_s_latitude,
20554: 					`out_order_from_lng` as out_s_longitude,
20555: 					`out_order_passengers_count` as out_passengers,
20556: 					`out_order_luggage_count` as out_luggage,
20557: 					`json` as u_details,
20558: 					(SELECT
20559: 						GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
20560: 					 FROM `users_order_comment`
20561: 					 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
20562: 					) as b_comments,
20563: 					(SELECT
20564: 						GROUP_CONCAT(`users_service`.`id_service` SEPARATOR ',')
20565: 					 FROM `users_service`
20566: 					 WHERE `users_service`.`id_user` = `users`.`id_user`
20567: 					) as b_services,
20568: 					(SELECT
20569: 						GROUP_CONCAT(
20570: 							CONCAT_WS('|',`users_order_location`.`id_order_location`,`users_order_location`.`basic`) 
20571: 						SEPARATOR ';')
20572: 					 FROM `users_order_location`
20573: 					 WHERE `users_order_location`.`id_user` = `users`.`id_user`
20574: 					) as b_location_classes,
20575: 					`id_schedule` as sc_id
20576: 				FROM `users`
20577: 				WHERE
20578: 					`users`.`deleted` = '0' AND `id_user_upper` = '" . $id_user . "'
20579: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
20580: 				";
20581: 
20582: 			$q = query($s);
20583: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
20584: 			
20585: 			$out = array('user' => array());
20586: 			while ($d = fetch_assoc($q))
20587: 			{
20588: 				$d['u_ban'] = get_user_ban_status($d['u_id']);
20589: 				$d['u_photo'] = $d['u_photo'] ? url($d['u_photo'],FILE_PATH) : '';
20590: 				add_time_zone($d['out_est_datetime']);
20591: 				if (!empty($d['b_comments'])) $d['b_comments'] = explode(',',$d['b_comments']);		
20592: 				if (!empty($d['b_services'])) $d['b_services'] = explode(',',$d['b_services']);			
20593: 				if (!empty($d['b_location_classes']))
20594: 				{
20595: 					$d['b_location_classes'] = explode(';',$d['b_location_classes']);
20596: 					foreach ($d['b_location_classes'] as $key=>$value)
20597: 					{
20598: 						$d['b_location_classes'][$key] = array();
20599: 						list(
20600: 							$d['b_location_classes'][$key]['b_location_class'],
20601: 							$d['b_location_classes'][$key]['basic']
20602: 							)= explode('|',$value);
20603: 						$d['b_location_classes'][$key]['basic'] = (int)$d['b_location_classes'][$key]['basic'];
20604: 					}
20605: 				}
20606: 				if (isset($d['u_phone_checked'])) $d['u_phone_checked'] = (int)$d['u_phone_checked'];
20607: 				$d['u_active'] = (int)$d['u_active'];			
20608: 				$d['out_drive'] = (int)$d['out_drive'];	
20609: 				$d['u_details'] = json_decode($d['u_details'],true);
20610: 				$out['user'][$d['u_id']] = $d;
20611: 			}
20612: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
20613: 			return array(
20614: 				'code' 		=>	'200',
20615: 				'status' 	=>	'success',		
20616: 				'data' 		=>	$out,
20617: 				'auth_user' =>	array(
20618: 									'u_id' => $_SESSION[UID],
20619: 									'u_name' => $_SESSION['name'],
20620: 									'u_family' => $_SESSION['family'],
20621: 									'u_middle' => $_SESSION['middle'],
20622: 									'u_email' => $_SESSION['email'],
20623: 									'u_phone' => $_SESSION['phone'],
20624: 									'u_role' => $_SESSION['id_role'],
20625: 									'u_a_role' => $this->id_role,
20626: 									'u_check_state' => $_SESSION['id_verification_status'],
20627: 									'u_ban' => $_SESSION['user_ban_status'],
20628: 									'u_active' => $_SESSION['active'],
20629: 									'u_photo' => $_SESSION['photo_link'],
20630: 									'u_birthday' => $_SESSION['birthday_date'],
20631: 									'u_lang' => $_SESSION['id_lang'],
20632: 									'u_currency' => $_SESSION['currency'],
20633: 									'u_gps_software' => $_SESSION['id_navigation']
20634: 								)
20635: 			);
20636: 		}
```

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 4=Administrator, 5=Agent`
- source range: `7526-8156`
```php
7526: 		public function selectOrder($id_order = "", $fields = 0)
7527: 		{
7528: 			if (empty($_SESSION[UID])) {
7529: 				return $this->showError('404', 'error', 'unauthorized access');
7530: 			}
7531: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
7532: 			{
7533: 				return $this->showError('404', 'error', 'wrong user role');
7534: 			}			
7535: 
7536: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = '';
7537: 			if (empty($id_order))
7538: 			{
7539: 				$sql_where = '1=1';
7540: 			}
7541: 			else
7542: 			{
7543: 				$sql_where = "`order`.`id_order` in (" . $id_order . ")";
7544: 			}
7545: 
7546: 			$field_flag = array();
7547: 			if (!empty($fields))
7548: 			{
7549: 				$field_arr	= get_field_arr('order',$this->id_role);
7550: 				$bin_arr = get_bin_arr();
7551: 
7552: 				foreach(str_split($fields) as $index => $char)
7553: 				{
7554: 					$value = get_0_64($char);
7555: 					if (empty($value)) continue;
7556: 					foreach($bin_arr as $bin_i)
7557: 					{
7558: 						if ($value & $bin_i) 
7559: 						{
7560: 							if (isset($field_arr[$index][$bin_i]))
7561: 							{
7562: 								$field_flag[$field_arr[$index][$bin_i]] = true;
7563: 							}
7564: 						}
7565: 					}
7566: 				}
7567: 			}
7568: 			if (!empty($field_flag['b_offers']))
7569: 			{
7570: 				$sql_order .= "(SELECT
7571: 						GROUP_CONCAT(
7572: 							CONCAT_WS('|',
7573: 								`id_user`,
7574: 								`create_datetime`
7575: 							)
7576: 						SEPARATOR ';') 
7577: 					 FROM `order_driver_select`
7578: 					 WHERE `order_driver_select`.`id_order` = `order`.`id_order`
7579: 					 ORDER BY `order_driver_select`.`id_user`
7580: 					) as b_offers,";
7581: 			}
7582: 			if (!empty($field_flag['b_offer']))
7583: 			{
7584: 				$sql_order .= "(SELECT
7585: 						COUNT(`id_user`)
7586: 					 FROM `order_driver_select`
7587: 					 WHERE 
7588: 						`order_driver_select`.`id_order` = `order`.`id_order` AND
7589: 						`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "'
7590: 					 LIMIT 1
7591: 					) as b_offer,";
7592: 			}
7593: 
7594: 			if ($this->id_role == 1 || $this->id_role == 4 || $this->id_role == 5)
7595: 			{
7596: 				$sql_order .= "
7597: 					`order`.`id_payment_method` as b_payment_way,
7598: 					`order`.`id_payment_card` as b_payment_card,
7599: 					`order`.`confirm_limit_datetime` as b_confirmation_limit,
7600: 					`order`.`confirm_datetime` as b_confirmation_datetime,
7601: 					`order`.`sum` as b_payment_sum,
7602: 					`order`.`pay_datetime` as b_payment_datetime,
7603: 					`order`.`code` as b_driver_code,
7604: 					(SELECT
7605: 						GROUP_CONCAT(
7606: 							CONCAT_WS('|',
7607: 								`id_user`,
7608: 								`datetime`
7609: 							)
7610: 						SEPARATOR ';') 
7611: 					 FROM `order_driver_attempt`
7612: 					 WHERE `order_driver_attempt`.`id_order` = `order`.`id_order`
7613: 					 ORDER BY `order_driver_attempt`.`id_user`
7614: 					) as b_attempts,
7615: 					";
7616: 				$sql_c_options .= "`order_driver`.`options`";
7617: 			}
7618: 			elseif ($this->id_role == 2)
7619: 			{
7620: 				$sql_order .= "
7621: 					(SELECT
7622: 							GROUP_CONCAT(
7623: 							CONCAT_WS('|',
7624: 								`id_user`,
7625: 								`datetime`
7626: 							)
7627: 						SEPARATOR ';') 
7628: 					 FROM `order_driver_attempt`
7629: 					 WHERE 
7630: 						`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
7631: 						`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
7632: 					) as b_attempts,
7633: 					od.`id_order_driver_status` as c_state,
7634: 					";	
7635: 				$sql_c_options .= "IF(`order_driver`.`id_user` = '" . $_SESSION[UID] . "',
7636: 									`order_driver`.`options`,0x02)";
7637: 			}
7638: 
7639: 			if ($this->id_role == 2 || $this->id_role == 4)
7640: 			{
7641: 				$sql_order_driver .= "
7642: 							`order_driver`.`id_payment_method`,
7643: 							IFNULL(`order_driver`.`id_payment_card`,0x02),
7644: 							IFNULL(`order_driver`.`sum`,0x02),
7645: 							`order_driver`.`pay_datetime`,
7646: 							";
7647: 			}
7648: 
7649: 			if ($this->id_role == 2)
7650: 			{
7651: 				$sql_left_join .= "
7652: 					LEFT JOIN (
7653: 							SELECT
7654: 								`id_order`,
7655: 								`id_user`,
7656: 								`id_order_driver_status`
7657: 							FROM
7658: 								`order_driver`
7659: 							WHERE
7660: 								`id_user` = '" . $_SESSION[UID] . "'
7661: 						) od USING (`id_order`)";
7662: 				$sql_where .= " AND ((`order`.`id_order_status` = '1' AND `order`.`max_waiting_datetime` > now() AND od.`id_user` IS NULL AND
7663: 					(SELECT
7664: 							COUNT(`order_driver_attempt`.`id_order`)
7665: 						FROM `order_driver_attempt`
7666: 						WHERE 
7667: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
7668: 							`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
7669: 					) < " . $this->constant['driver_code_attempt_count_max'] . ") OR 
7670: 					od.`id_user` = '" . $_SESSION[UID] . ("' OR (SELECT
7671: 							`order_driver_select`.`cancel`
7672: 						FROM `order_driver_select`
7673: 						WHERE 
7674: 							`order_driver_select`.`id_order` = `order`.`id_order` AND 
7675: 							`order_driver_select`.`id_user` = '" . $_SESSION[UID] . "' 
7676: 					) IS NOT NULL") . ") AND `order`.`client` != '" . $_SESSION[UID] . "'";	
7677: 			}
7678: 			elseif ($this->id_role == 1 || $this->id_role == 5)
7679: 			{
7680: 				$sql_where .= " AND `order`.`client` = '" . $_SESSION[UID] . "'";
7681: 			}
7682: 			if (DEFAULT_PROFILE == 'stadium')
7683: 			{
7684: 				$sql_order .= "(SELECT
7685: 							GROUP_CONCAT(CONCAT_WS(0x00,`id_seat`,`id_trip`,`id_trip_seat`) SEPARATOR 0x01)
7686: 						 FROM
7687: 							`ticket`
7688: 						 WHERE `ticket`.`id_order` = `order`.`id_order`
7689: 						) as fix_seats,";
7690: 			}
7691: 
7692: 			$s = "SELECT
7693: 					`order`.`id_order` as b_id,
7694: 					`order`.`client` as u_id,
7695: 					`order`.`from` as b_start_address,
7696: 					`order`.`from_lat` as b_start_latitude,
7697: 					`order`.`from_lng` as b_start_longitude,
7698: 					`order`.`to` as b_destination_address,
7699: 					`order`.`to_lat` as b_destination_latitude,
7700: 					`order`.`to_lng` as b_destination_longitude,
7701: 					`order`.`datetime_start_plan` as b_start_datetime,
7702: 					`order`.`comment` as b_custom_comment,
7703: 					`order`.`flight_number` as b_flight_number,
7704: 					`order`.`terminal` as b_terminal,
7705: 					`order`.`passenger_count` as b_passengers_count,
7706: 					`order`.`luggage_count` as b_luggage_count,
7707: 					`order`.`placard` as b_placard,
7708: 					`order`.`id_car_class` as b_car_class,
7709: 					" . $sql_order . "
7710: 					`order`.`tips` as b_tips,
7711: 					`order`.`id_order_status` as b_state,
7712: 					`order`.`rating` as b_rating,
7713: 					`order`.`create_datetime` as b_created,
7714: 					`order`.`is_confirmed` as b_confirm_state,
7715: 					`order`.`car_count` as b_cars_count,
7716: 					`order`.`cancel_reason` as b_cancel_reason,
7717: 					(SELECT
7718: 						GROUP_CONCAT(
7719: 							CONCAT_WS('|',
7720: 								`id_user`,
7721: 								`id_order_cancel`
7722: 							)
7723: 						SEPARATOR ';')
7724: 						FROM `order_cancel_items`
7725: 						WHERE `order_cancel_items`.`id_order` = `order`.`id_order`
7726: 						ORDER BY `id_user`
7727: 					) as b_cancel_states,
7728: 					`order`.`approve_datetime` as b_approved,
7729: 					`order`.`cancel_datetime` as b_canceled,
7730: 					`order`.`complete_datetime` as b_completed,
7731: 					`order`.`max_waiting_datetime` as b_max_waiting,
7732: 					(SELECT
7733: 						GROUP_CONCAT(
7734: 							CONCAT_WS('|',
7735: 								`interval`,
7736: 								`datetime`,
7737: 								`index`
7738: 							)
7739: 						SEPARATOR ';') 
7740: 					 FROM `order_waiting_time`
7741: 					 WHERE `order_waiting_time`.`id_order` = `order`.`id_order`
7742: 					) as b_max_waiting_list,				
7743: 					`order`.`estimated_waiting_datetime` as b_estimate_waiting,
7744: 					`order`.`options` as b_options,
7745: 					`order`.`contact` as b_contact,
7746: 					`order`.`id_order_location` as b_location_class,
```

### `selectPayment` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `24864-24965`
```php
24864: 		public function selectPayment($id_payment = '', $payment_services = array(), $payment_ways = array())
24865: 		{
24866: 			if (empty($_SESSION[UID])) {
24867: 				return $this->showError('404', 'error', 'unauthorized access');
24868: 			}
24869: 
24870: 			$s = "SELECT
24871: 					`id_payment` as p_id,
24872: 					`sum`,
24873: 					`percent`,
24874: 					`total_sum`,
24875: 					`currency`,
24876: 					`json`,
24877: 					`id_payment_status` as payment_status,
24878: 					`id_payment_service` as payment_service,
24879: 					`from`,
24880: 					`to`,
24881: 					`create_user` as c_u_id,
24882: 					`last_edit_user` as e_u_id,
24883: 					`last_edit_int_timestamp` as timestamp_edited,
24884: 					`create_int_timestamp` as timestamp_created,
24885: 					`id_user_subscription` as subs_id,
24886: 					`id_payment_method` as payment_way,
24887: 					`id_outer` as p_id_outer					
24888: 				FROM `payment`
24889: 				WHERE
24890: 					`id_payment` = '$id_payment'
24891: 				";
24892: 
24893: 			$q = query($s);
24894: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
24895: 
24896: 			$out = array('payment' => array());
24897: 			$arr = array(
24898: 					'yookassa'	=>	array(
24899: 							'pending'				=>		'1',
24900: 							'waiting_for_capture'	=>		'2',
24901: 							'succeeded'				=>		'6',
24902: 							'canceled'				=>		'3'
24903: 					)			
24904: 			);
24905: 			$update_arr = array();
24906: 			while ($d = fetch_assoc($q))
24907: 			{
24908: 				if ($d['from'] != $_SESSION[UID])
24909: 				{
24910: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24911: 				}
24912: 				$d['json'] = json_decode($d['json'],true);
24913: 				if (!in_array($d['payment_status'],array(3,4,5,6,7)))
24914: 				{
24915: 					if (!empty($payment_services[$d['payment_service']]['var']))
24916: 					{
24917: 						$f_name = "{$payment_services[$d['payment_service']]['var']}_get_payment";
24918: 						$payment_status = $f_name($d['p_id_outer']);
24919: 						if (!empty($payment_status['error'])) return $this->showError('404', 'error', $payment_status['error']);
24920: 						$payment_status_int = $arr[$payment_services[$d['payment_service']]['var']][$payment_status['status']];
24921: 						if ($d['payment_status'] != $payment_status_int)
24922: 						{
24923: 							$d['payment_status'] = $payment_status_int;
24924: 							$d['json']['status_str'] = $payment_status['status'];
24925: 							$d['json']['paid_str'] = var_export($payment_status['paid'],true);
24926: 							$update_arr[] = array($payment_status_int,real_escape_string(json_encode($d['json'])),$d['p_id']);
24927: 						}	
24928: 					}
24929: 				}					
24930: 				$out['payment'][] = $d;
24931: 			}
24932: 			if (!empty($update_arr))
24933: 			{
24934: 				foreach($update_arr as $upd)
24935: 				{
24936: 					$s = "UPDATE `payment`
24937: 						SET
24938: 							`json` = '{$upd[1]}',
24939: 							`id_payment_status` = '{$upd[0]}',
24940: 							`last_edit_user` = '{$_SESSION[UID]}',
24941: 							`last_edit_int_timestamp` =  '" . time() . "'
24942: 						WHERE
24943: 							`id_payment` = '{$upd[2]}'
24944: 						";
24945: 
24946: 					$q = query($s);
24947: 						
24948: 					$update = affected_rows();
24949: 					if ($update === -1) 
24950: 					{
24951: 						return $this->showError('400','error','update failed'. error_db());
24952: 					}
24953: 					elseif ($update === 0) 
24954: 					{
24955: 						return $this->showError('400','error','modified data not found');
24956: 					}
24957: 				}
24958: 			}
24959: 
24960: 			return array(
24961: 				'code' 		=>	'200',
24962: 				'status' 	=>	'success',		
24963: 				'data' 		=>	$out
24964: 			);
24965: 		}
```

### `selectProcessingOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `4943-5601`
```php
4943: 		public function selectProcessingOrder($fields = 0, $filter = NULL, $order_location = array())
4944: 		{
4945: 			if (empty($_SESSION[UID])) {
4946: 				return $this->showError('404', 'error', 'unauthorized access');
4947: 			}
4948: 			if ($this->id_role != 2 && $this->id_role != 4)
4949: 			{
4950: 				return $this->showError('404', 'error', 'wrong user role');
4951: 			}
4952: 
4953: 			$sql_order = $sql_drivers = $sql_order_driver = $sql_c_options = $sql_payment = $sql_left_join = $sql_where_order = $sql_where = '';
4954: 		
4955: 			$field_flag = array();
4956: 			if (!empty($fields))
4957: 			{
4958: 				$field_arr	= get_field_arr('processingOrder',$this->id_role);
4959: 				$bin_arr = get_bin_arr();
4960: 
4961: 				foreach(str_split($fields) as $index => $char)
4962: 				{
4963: 					$value = get_0_64($char);
4964: 					if (empty($value)) continue;
4965: 					foreach($bin_arr as $bin_i)
4966: 					{
4967: 						if ($value & $bin_i) 
4968: 						{
4969: 							if (isset($field_arr[$index][$bin_i]))
4970: 							{
4971: 								$field_flag[$field_arr[$index][$bin_i]] = true;
4972: 							}
4973: 						}
4974: 					}
4975: 				}
4976: 			}
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
5080: 						(SELECT
5081: 							GROUP_CONCAT(`users_service`.`id_service` SEPARATOR ',')
5082: 						 FROM `users_service`
5083: 						 WHERE `users_service`.`id_user` = `users`.`id_user`
5084: 						 ORDER BY `users_service`.`id_service`
5085: 						) as b_services,
5086: 						(SELECT
5087: 							GROUP_CONCAT(`users_order_location`.`id_order_location` SEPARATOR ',')
5088: 						 FROM `users_order_location`
5089: 						 WHERE `users_order_location`.`id_user` = `users`.`id_user`
5090: 						) as b_location_classes,						
5091: 						IF(`taxi_license`.`id_car` IS NULL,NULL,
5092: 							GROUP_CONCAT(IF(`taxi_license`.`active` = '1' AND `taxi_license_order_location`.`id_order_location` IS NOT NULL,
5093: 								CONCAT_WS('|',
5094: 									`taxi_license_order_location`.`id_order_location`,
5095: 									`taxi_license_order_location`.`location_table_column_value`
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
5108: 					LIMIT 1
5109: 					";				
5110: 
5111: 				$q = query($s);
5112: 				if ($q === false) return $this->showError('404', 'error', 'user select failed');
5113: 				$d = fetch_assoc($q);
5114: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'used car not found');
5115: 				if (empty($d['b_location_classes'])) return $this->showError('404', 'error', 'empty driver location classes');
5116: 				$license = array();
5117: 				if (!empty($d['license']))
5118: 				{
5119: 					$d['license'] = explode(';',$d['license']);
5120: 					foreach ($d['license'] as $key=>$value)
5121: 					{
5122: 						$d['license'][$key] = array();
5123: 						list(
5124: 							$d['license'][$key]['id_order_location'],
5125: 							$d['license'][$key]['location_table_column_value']
5126: 							)= explode('|',$value);
5127: 					}
5128: 					$license = $d['license'];
5129: 				}
5130: 				$sql_where .= " AND (`order`.`id_car_class` IS NULL" . (empty($d['id_car_class']) ? "" : " OR `order`.`id_car_class` = '" . $d['id_car_class'] . "'") . ") AND `order`.`id_order_location` in (" . $d['b_location_classes'] . ")";
5131: 				$sql_having = "HAVING (b_comments IS NULL" . (empty($d['b_comments']) ? "" : " OR '" . $d['b_comments'] . "' RLIKE CONCAT_WS('','(^|,)',REPLACE(b_comments,',',',([0-9],)*'),'(,|$)')") . ") AND (b_services IS NULL" . (empty($d['b_services']) ? "" : " OR '" . $d['b_services'] . "' RLIKE CONCAT_WS('','(^|,)',REPLACE(b_services,',',',([0-9],)*'),'(,|$)')") . ")";
5132: 				if (isset($d['lat']) && isset($d['lng']))
5133: 				{
5134: 					$union = true;
5135: 					$r_earth = 6371000; //м
5136: 					$lat_5km_in_degrees = 5000*180/$r_earth/pi();
5137: 					$lng_5km_in_degrees = 5000*180/$r_earth/pi()/cos(pi()/180*$d['lat']);
5138: 					$sql_where_union_0 = " AND `order`.`from_lat` AND `order`.`from_lng` AND `order`.`from_lat` < '" . $d['lat'] . "' + '" . $lat_5km_in_degrees . "' AND `order`.`from_lat` > '" . $d['lat'] . "' - '" . $lat_5km_in_degrees . "' AND `order`.`from_lng` < '" . $d['lng'] . "' + '" . $lng_5km_in_degrees . "' AND `order`.`from_lng` > '" . $d['lng'] . "' - '" . $lng_5km_in_degrees . "'";
5139: 					$lat_15km_in_degrees = $lat_5km_in_degrees*3;
5140: 					$lng_15km_in_degrees = $lng_5km_in_degrees*3;
5141: 					$sql_where_union_1 = " AND `order`.`from_lat` AND `order`.`from_lng` AND (((`order`.`from_lat` >= '" . $d['lat'] . "' + '" . $lat_5km_in_degrees . "' AND `order`.`from_lat` < '" . $d['lat'] . "' + '" . $lat_15km_in_degrees . "') OR (`order`.`from_lat` <= '" . $d['lat'] . "' - '" . $lat_5km_in_degrees . "' AND `order`.`from_lat` > '" . $d['lat'] . "' - '" . $lat_15km_in_degrees . "')) OR ((`order`.`from_lng` >= '" . $d['lng'] . "' + '" . $lng_5km_in_degrees . "' AND `order`.`from_lng` < '" . $d['lng'] . "' + '" . $lng_15km_in_degrees . "') OR (`order`.`from_lng` <= '" . $d['lng'] . "' - '" . $lng_5km_in_degrees . "' AND `order`.`from_lng` > '" . $d['lng'] . "' - '" . $lng_15km_in_degrees . "')))";
5142: 					$sql_where_union_2 = " AND `order`.`from_lat` IS NULL OR `order`.`from_lng` IS NULL OR ((`order`.`from_lat` >= '" . $d['lat'] . "' + '" . $lat_15km_in_degrees . "' OR `order`.`from_lat` <= '" . $d['lat'] . "' - '" . $lat_15km_in_degrees . "') || (`order`.`from_lng` >= '" . $d['lng'] . "' + '" . $lng_15km_in_degrees . "' OR `order`.`from_lng` <= '" . $d['lng'] . "' - '" . $lng_15km_in_degrees . "'))";
5143: 					$sql_distance = ",
5144: 					2*'" . $r_earth . "'*ASIN(SQRT(POW(SIN(PI()/360*(`order`.`from_lat` - '" . $d['lat'] . "')),2) + COS(PI()/360*'" . $d['lat'] . "')*COS(PI()/360*`order`.`from_lat`)*POW(SIN(PI()/360*(`order`.`from_lng` - '" . $d['lng'] . "')),2))) as b_co_distance,
5145: 					TIMESTAMPDIFF(SECOND,now(),`order`.`max_waiting_datetime`) as b_co_time";
5146: 					$sql_having .= " AND b_co_distance <= b_co_time*'" . $this->constant['average_speed'] . "'/3.6";
5147: 					$sql_order_by = "ORDER BY `order`.`max_waiting_datetime`";
5148: /*	MAX(ABS())		DEGREES(PI())	180			RADIANS(180)	3.141592653689793	*/	
5149: 				}
5150: 			}
5151: 
5152: 			$booking = array();
5153: 			$s = "SELECT
5154: 					`order`.`id_order` as b_id,
5155: 					" . $sql_order . "
5156: 					`order`.`client` as u_id,
5157: 					`order`.`from` as b_start_address,
5158: 					`order`.`from_lat` as b_start_latitude,
5159: 					`order`.`from_lng` as b_start_longitude,
5160: 					`order`.`to` as b_destination_address,
5161: 					`order`.`to_lat` as b_destination_latitude,
5162: 					`order`.`to_lng` as b_destination_longitude,
5163: 					`order`.`datetime_start_plan` as b_start_datetime,
```

### `selectReferralUrl` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `10921-10995`
```php
10921: 		public function selectReferralUrl($id_user = "")
10922: 		{
10923: 			if (empty($_SESSION[UID])) {
10924: 				return $this->showError('404', 'error', 'unauthorized access');
10925: 			}
10926: 
10927: 			$sql_limit = "LIMIT 1";
10928: 
10929: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10930: 			{
10931: 				$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
10932: 			}
10933: 			else
10934: 			{	
10935: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10936: 
10937: 				$sql_where = "AND `id_user` in (" . $id_user . ")";
10938: 
10939: 				$sql_limit = "LIMIT " . $this->limit_offset . ", " . $this->limit_row_count;
10940: 			}
10941: 
10942: 			$s = "SELECT 
10943: 					`id_user` as u_id
10944: 				FROM `users`
10945: 				WHERE
10946: 					`deleted` = '0'
10947: 				" . $sql_where . "
10948: 				" . $sql_limit . "
10949: 				";
10950: 
10951: 			$q = query($s);
10952: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
10953: 			
10954: 			$out = array('user' => array());
10955: 
10956: 			while ($d = fetch_assoc($q))
10957: 			{
10958: 				$path = url('app/' . get_path_from_user($d['u_id']),CONFIG_URL);
10959: 				$d['url'] = array(								
10960: 								'register'	=>	$path . '/register/',
10961: 								'download'	=>	$path . '/download/'
10962: 							);
10963: 
10964: 				$out['user'][$d['u_id']] = $d;
10965: 			}
10966: 
10967: 
10968: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
10969: 
10970: 
10971: 
10972: 			return array(
10973: 				'code' 		=>	'200',
10974: 				'status' 	=>	'success',		
10975: 				'data' 		=>	$out,
10976: 				'auth_user' =>	array(
10977: 									'u_id' => $_SESSION[UID],
10978: 									'u_name' => $_SESSION['name'],
10979: 									'u_family' => $_SESSION['family'],
10980: 									'u_middle' => $_SESSION['middle'],
10981: 									'u_email' => $_SESSION['email'],
10982: 									'u_phone' => $_SESSION['phone'],
10983: 									'u_role' => $_SESSION['id_role'],
10984: 									'u_a_role' => $this->id_role,
10985: 									'u_check_state' => $_SESSION['id_verification_status'],
10986: 									'u_ban' => $_SESSION['user_ban_status'],
10987: 									'u_active' => $_SESSION['active'],
10988: 									'u_photo' => $_SESSION['photo_link'],
10989: 									'u_birthday' => $_SESSION['birthday_date'],
10990: 									'u_lang' => $_SESSION['id_lang'],
10991: 									'u_currency' => $_SESSION['currency'],
10992: 									'u_gps_software' => $_SESSION['id_navigation']
10993: 								)
10994: 			);
10995: 		}
```

### `selectReferralUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `11066-11199`
```php
11066: 		public function selectReferralUser($id_user = "")
11067: 		{
11068: 			if (empty($_SESSION[UID])) {
11069: 				return $this->showError('404', 'error', 'unauthorized access');
11070: 			}
11071: 			$sql_add = "
11072: 					`phone` as u_phone,
11073: 					`phone_is_verified` as u_phone_checked,
11074: 					`email` as u_email,
11075: 					`birthday_date` as u_birthday,
11076: 					`referral_code` as ref_code,
11077: 					`referrer` as referrer_u_id,
11078: 					`tg` as u_tg,
11079: 					`tg_is_verified` as u_tg_checked,
11080: 					`id_user_upper` as u_upper,
11081: 					`wa` as u_wa,
11082: 					`wa_is_verified` as u_wa_checked,";
11083: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
11084: 			{
11085: 				$id_user = $_SESSION[UID];
11086: 				if ($this->id_role != 4) $sql_add = "";
11087: 			}
11088: 			else
11089: 			{	
11090: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11091: 			}
11092: 
11093: 			$s = "SELECT 
11094: 					`id_user` as u_id,
11095: 					`id_role` as u_role,
11096: 					`name` as u_name,
11097: 					`family` as u_family,
11098: 					`middle` as u_middle,
11099: 					" . $sql_add . "
11100: 					`photo_link` as u_photo,
11101: 					`id_lang` as u_lang,
11102: 					`currency` as u_currency,
11103: 					`id_city` as u_city,
11104: 					`tips` as u_tips,
11105: 					`language_skills` as u_lang_skills,
11106: 					`description` as u_description,
11107: 					`id_navigation` as u_gps_software,
11108: 					`id_verification_status` as u_check_state,
11109: 					`active` as u_active,			
11110: 					`out_order` as out_drive,
11111: 					`out_order_to` as out_address,
11112: 					`out_order_to_lat` as out_latitude,
11113: 					`out_order_to_lng` as out_longitude,
11114: 					`out_order_complete_datetime` as out_est_datetime,
11115: 					`out_order_from` as out_s_address,
11116: 					`out_order_from_lat` as out_s_latitude,
11117: 					`out_order_from_lng` as out_s_longitude,
11118: 					`out_order_passengers_count` as out_passengers,
11119: 					`out_order_luggage_count` as out_luggage,
11120: 					`json` as u_details,
11121: 					(SELECT
11122: 						GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
11123: 					 FROM `users_order_comment`
11124: 					 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
11125: 					) as b_comments,
11126: 					(SELECT
11127: 						GROUP_CONCAT(`users_service`.`id_service` SEPARATOR ',')
11128: 					 FROM `users_service`
11129: 					 WHERE `users_service`.`id_user` = `users`.`id_user`
11130: 					) as b_services,
11131: 					(SELECT
11132: 						GROUP_CONCAT(
11133: 							CONCAT_WS('|',`users_order_location`.`id_order_location`,`users_order_location`.`basic`) 
11134: 						SEPARATOR ';')
11135: 					 FROM `users_order_location`
11136: 					 WHERE `users_order_location`.`id_user` = `users`.`id_user`
11137: 					) as b_location_classes,
11138: 					`id_schedule` as sc_id
11139: 				FROM `users`
11140: 				WHERE
11141: 					`users`.`deleted` = '0' AND `referrer` = '" . $id_user . "'
11142: 				LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
11143: 				";
11144: 
11145: 			$q = query($s);
11146: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
11147: 			
11148: 			$out = array('user' => array());
11149: 			while ($d = fetch_assoc($q))
11150: 			{
11151: 				$d['u_ban'] = get_user_ban_status($d['u_id']);
11152: 				$d['u_photo'] = $d['u_photo'] ? url($d['u_photo'],FILE_PATH) : '';
11153: 				add_time_zone($d['out_est_datetime']);
11154: 				if (!empty($d['b_comments'])) $d['b_comments'] = explode(',',$d['b_comments']);		
11155: 				if (!empty($d['b_services'])) $d['b_services'] = explode(',',$d['b_services']);			
11156: 				if (!empty($d['b_location_classes']))
11157: 				{
11158: 					$d['b_location_classes'] = explode(';',$d['b_location_classes']);
11159: 					foreach ($d['b_location_classes'] as $key=>$value)
11160: 					{
11161: 						$d['b_location_classes'][$key] = array();
11162: 						list(
11163: 							$d['b_location_classes'][$key]['b_location_class'],
11164: 							$d['b_location_classes'][$key]['basic']
11165: 							)= explode('|',$value);
11166: 						$d['b_location_classes'][$key]['basic'] = (int)$d['b_location_classes'][$key]['basic'];
11167: 					}
11168: 				}
11169: 				if (isset($d['u_phone_checked'])) $d['u_phone_checked'] = (int)$d['u_phone_checked'];
11170: 				$d['u_active'] = (int)$d['u_active'];			
11171: 				$d['out_drive'] = (int)$d['out_drive'];	
11172: 				$d['u_details'] = json_decode($d['u_details'],true);
11173: 				$out['user'][$d['u_id']] = $d;
11174: 			}
11175: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
11176: 			return array(
11177: 				'code' 		=>	'200',
11178: 				'status' 	=>	'success',		
11179: 				'data' 		=>	$out,
11180: 				'auth_user' =>	array(
11181: 									'u_id' => $_SESSION[UID],
11182: 									'u_name' => $_SESSION['name'],
11183: 									'u_family' => $_SESSION['family'],
11184: 									'u_middle' => $_SESSION['middle'],
11185: 									'u_email' => $_SESSION['email'],
11186: 									'u_phone' => $_SESSION['phone'],
11187: 									'u_role' => $_SESSION['id_role'],
11188: 									'u_a_role' => $this->id_role,
11189: 									'u_check_state' => $_SESSION['id_verification_status'],
11190: 									'u_ban' => $_SESSION['user_ban_status'],
11191: 									'u_active' => $_SESSION['active'],
11192: 									'u_photo' => $_SESSION['photo_link'],
11193: 									'u_birthday' => $_SESSION['birthday_date'],
11194: 									'u_lang' => $_SESSION['id_lang'],
11195: 									'u_currency' => $_SESSION['currency'],
11196: 									'u_gps_software' => $_SESSION['id_navigation']
11197: 								)
11198: 			);
11199: 		}
```

### `selectTask` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `22926-23043`
```php
22926: 		public function selectTask($id_task_list = '')
22927: 		{
22928: 			if (empty($_SESSION[UID])) {
22929: 				return $this->showError('404', 'error', 'unauthorized access');
22930: 			}
22931: 
22932: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22933: 
22934: 			$sql_where = '1 = 1';
22935: 			if (!empty($id_task_list))
22936: 			{
22937: 				$sql_where = "`id_task_list` in ($id_task_list)";
22938: 			}
22939: 			$s = "SELECT 
22940: 				  `id_task_list` as tl_id,
22941: 				  `id_task_status` as task_status,
22942: 				  `min_action_interval`,
22943: 				  `max_action_interval`,
22944: 				  `comment`,
22945: 				  `account`,
22946: 				  `json`,
22947: 				  `active`,
22948: 				  `last_edit_datetime` as tl_edited,
22949: 				  `create_datetime` as tl_created,	  
22950: 				 (SELECT
22951: 					GROUP_CONCAT(`id_account` SEPARATOR ',')
22952: 				  FROM `task_list_account`
22953: 				  WHERE `task_list_account`.`id_task_list` = `task_list`.`id_task_list`
22954: 				 ) as bind_account,
22955: 				 (SELECT
22956: 					GROUP_CONCAT(`id_task_comment` SEPARATOR ',')
22957: 				  FROM `task_list_comment`
22958: 				  WHERE `task_list_comment`.`id_task_list` = `task_list`.`id_task_list`
22959: 				 ) as bind_comment, 
22960: 				(SELECT
22961: 					IF(`id_task_action_function` IS NULL,NULL,
22962: 						GROUP_CONCAT(
22963: 							CONCAT_WS(0x00,
22964: 								`id_task_action_function`,
22965: 								`parameter`
22966: 							)
22967: 						SEPARATOR 0x01)
22968: 					)
22969: 				 FROM `task_list_action_function`
22970: 				 WHERE `task_list_action_function`.`id_task_list` = `task_list`.`id_task_list`
22971: 				) as bind_action_function,	
22972: 				(SELECT
22973: 					IF(`id_task_action_control` IS NULL,NULL,
22974: 						GROUP_CONCAT(
22975: 							CONCAT_WS(0x00,
22976: 								`id_task_action_control`,
22977: 								UNIX_TIMESTAMP(`create_timestamp`),
22978: 								`complete`,
22979: 								`active`
22980: 							)
22981: 						SEPARATOR 0x01)
22982: 					)
22983: 				 FROM `task_list_action_control`
22984: 				 WHERE `task_list_action_control`.`id_task_list` = `task_list`.`id_task_list`
22985: 				) as bind_action_control
22986: 			FROM `task_list`
22987: 			WHERE
22988: 				$sql_where
22989: 			LIMIT " . $this->limit_offset . ", " . $this->limit_row_count . "
22990: 			";
22991: 
22992: 			$q = query($s);
22993: 			if ($q === false) return $this->showError('404', 'error', 'select failed');
22994: 
22995: 			$out = array('task list' => array());
22996: 			while ($d = fetch_assoc($q))
22997: 			{
22998: 				$d['comment'] = json_decode($d['comment'],true);
22999: 				$d['account'] = json_decode($d['account'],true);
23000: 				$d['json'] = json_decode($d['json'],true);
23001: 				add_time_zone($d['tl_edited']);
23002: 				add_time_zone($d['tl_created']);
23003: 				if (!empty($d['bind_account'])) $d['bind_account'] = explode(',',$d['bind_account']);
23004: 				if (!empty($d['bind_comment'])) $d['bind_comment'] = explode(',',$d['bind_comment']);
23005: 				if (!empty($d['bind_action_function']))
23006: 				{
23007: 					$d['bind_action_function'] = explode(chr(1),$d['bind_action_function']);
23008: 					foreach ($d['bind_action_function'] as $key=>$value)
23009: 					{
23010: 						$d['bind_action_function'][$key] = array();
23011: 						list(
23012: 							$d['bind_action_function'][$key]['task_action_function'],
23013: 							$d['bind_action_function'][$key]['parameter']
23014: 						)= explode(chr(0),$value);
23015: 						$d['bind_action_function'][$key]['parameter'] = json_decode($d['bind_action_function'][$key]['parameter'],true);
23016: 					}
23017: 				}
23018: 	
23019: 				if (!empty($d['bind_action_control']))
23020: 				{
23021: 					$d['bind_action_control'] = explode(chr(1),$d['bind_action_control']);
23022: 					foreach ($d['bind_action_control'] as $key=>$value)
23023: 					{
23024: 						$d['bind_action_control'][$key] = array();
23025: 						list(
23026: 							$d['bind_action_control'][$key]['task_action_control'],
23027: 							$d['bind_action_control'][$key]['unix_timestamp_created'],
23028: 							$d['bind_action_control'][$key]['complete'],
23029: 							$d['bind_action_control'][$key]['active']
23030: 						)= explode(chr(0),$value);
23031: 					}
23032: 				}
23033: 
23034: 				$out['task list'][$d['tl_id']] = $d;
23035: 			}
23036: 			if (empty($this->associativeArray)) $out['task list'] =  array_values($out['task list']);
23037: 
23038: 			return array(
23039: 				'code' 		=>	'200',
23040: 				'status' 	=>	'success',
23041: 				'data' 		=>	$out
23042: 			);
23043: 		}
```

### `selectTicket` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver`
- source range: `18936-19005`
```php
18936: 		public function selectTicket($sc_id = "")
18937: 		{	
18938: 			$s = "SELECT
18939: 					`id_schedule` as sc_id,
18940: 					`id_trip` as t_id,
18941: 					COUNT(IF(`id_order` IS NULL AND `status` NOT in (2,3),`id_schedule`,NULL)) as available,
18942: 					COUNT(IF(`id_order` IS NOT NULL OR `status` = 2,`id_schedule`,NULL)) as taken,
18943: 					COUNT(IF(
18944: 						(SELECT
18945: 							`product`
18946: 						FROM `cart`
18947: 						WHERE 
18948: 							`product` = `ticket`.`id_trip` AND `property` = `ticket`.`id_trip_seat` AND `booking_limit` > now()
18949: 						LIMIT 1
18950: 						) IS NULL,IF(`ticket`.`id_order` IS NULL AND `status` NOT in (2,3),`ticket`.`id_schedule`,NULL),NULL
18951: 					)) as full_available
18952: 				FROM `ticket`
18953: 				WHERE 
18954: 					 `id_schedule` in (" . $sc_id . ")
18955: 				GROUP BY `id_trip`
18956: 				";
18957: 
18958: 			if (empty($_SESSION[UID]) || $this->id_role != 2) {
18959: 				if (empty($sc_id)) return $this->showError('404', 'error', 'empty sc_id');
18960: 			}
18961: 			else
18962: 			{
18963: 				if (empty($sc_id)) 
18964: 				{
18965: 					$s = "SELECT
18966: 							`ticket`.`id_schedule` as sc_id,
18967: 							`ticket`.`id_trip` as t_id,
18968: 							COUNT(IF(`ticket`.`id_order` IS NULL AND `ticket`.`status` NOT in (2,3),`ticket`.`id_schedule`,NULL)) as available,
18969: 							COUNT(IF(`ticket`.`id_order` IS NOT NULL OR `ticket`.`status` = 2,`ticket`.`id_schedule`,NULL)) as taken,
18970: 							COUNT(IF(
18971: 								(SELECT
18972: 									`product`
18973: 								FROM `cart`
18974: 								WHERE 
18975: 									`product` = `ticket`.`id_trip` AND `property` = `ticket`.`id_trip_seat` AND `booking_limit` > now()
18976: 								LIMIT 1
18977: 								) IS NULL,IF(`ticket`.`id_order` IS NULL AND `ticket`.`status` NOT in (2,3),`ticket`.`id_schedule`,NULL),NULL
18978: 							)) as full_available
18979: 						FROM `ticket`
18980: 						LEFT JOIN `trip` USING (`id_trip`)
18981: 						WHERE 
18982: 							 `trip`.`driver` = '" . $_SESSION[UID] . "'
18983: 						GROUP BY `ticket`.`id_trip`
18984: 						";
18985: 				}
18986: 			}
18987: 
18988: 			$q = query($s);
18989: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
18990: 			
18991: 			$out = array('schedule' => array());
18992: 			while ($d = fetch_assoc($q))
18993: 			{
18994: 				if (empty($d['sc_id'])) continue;
18995: 				$d['available'] = (int)$d['available'];
18996: 				$d['taken'] = (int)$d['taken'];
18997: 				$out['schedule'][$d['sc_id']][] = $d;
18998: 			}
18999: 
19000: 			return array(
19001: 				'code' 		=>	'200',
19002: 				'status' 	=>	'success',
19003: 				'data'		=>	$out
19004: 			);	
19005: 		}
```

### `selectTrip` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 4=Administrator, 5=Agent`
- source range: `12130-12878`
```php
12130: 		public function selectTrip($id_trip = "", $type = NULL, $id_order = "", $filter = NULL, $fields = 0, $raw_price = false, $with_import = false, $price_time_functions = array(), $aggregators = array())
12131: 		{
12132: 			if (empty($_SESSION[UID])) {
12133: 				if ($type !== NULL) return $this->showError('404', 'error', 'unauthorized access');
12134: 			}
12135: 			else
12136: 			{
12137: 				if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
12138: 				{
12139: 					if ($type !== NULL) return $this->showError('404', 'error', 'wrong user role');
12140: 				}
12141: 			}
12142: 
12143: 			$sql_left_join = $orders = $cart = '';
12144: 			if (empty($id_trip))
12145: 			{
12146: 				$sql_where = '1=1';
12147: 			}
12148: 			else
12149: 			{
12150: 				$sql_where = "`trip`.`id_trip` in (" . $id_trip . ")";
12151: 			}
12152: 			if (empty($with_import) && empty($this->constant['select_trip_with_import'])) $sql_where .= ' AND `trip`.`id_aggregator` IS NULL';
12153: 			$cart_sold_seats = "";
12154: 			$field_flag = array();
12155: 			if (!empty($fields))
12156: 			{
12157: 				$field_arr	= get_field_arr(empty($type) ? 'trip' : "{$type}Trip",$this->id_role);
12158: 				$bin_arr = get_bin_arr();
12159: 				foreach(str_split($fields) as $index => $char)
12160: 				{
12161: 					$value = get_0_64($char);
12162: 					if (empty($value)) continue;
12163: 					foreach($bin_arr as $bin_i)
12164: 					{
12165: 						if ($value & $bin_i) 
12166: 						{
12167: 							if (isset($field_arr[$index][$bin_i]))
12168: 							{
12169: 								$field_flag[$field_arr[$index][$bin_i]] = true;
12170: 							}
12171: 						}
12172: 					}
12173: 				}
12174: 			}
12175: 			$sql_left_join_schedule = '';
12176: 			if (DEFAULT_PROFILE == 'stadium')
12177: 			{
12178: 				$cart_sold_seats = ",
12179: 					(SELECT
12180: 						GROUP_CONCAT(CONCAT_WS(0x00,`id_user`,`property`,`booking_limit`) SEPARATOR 0x01)
12181: 					 FROM `cart`
12182: 					 WHERE `product` = `trip`.`id_trip` AND `booking_limit` > now()
12183: 					) as cart_seats,
12184: 					(SELECT
12185: 						GROUP_CONCAT(CONCAT_WS(0x00,`ticket`.`id_seat`,`ticket`.`id_trip_seat`,IFNULL(`ticket`.`tariff`,0x02),IFNULL(`ticket`.`currency`,0x02),IFNULL(`order`.`client`,0x02),IFNULL(`order`.`id_order`,0x02),IF(`order`.`pay_datetime`,'',IFNULL(`order`.`confirm_limit_datetime`,0x02)),IFNULL(`order`.`pay_datetime`,0x02),IFNULL(`ticket`.`number`,0x02),`status`) SEPARATOR 0x01)
12186: 					FROM `ticket`
12187: 					LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`
12188: 					WHERE 
12189: 						 `ticket`.`id_trip` = `trip`.`id_trip`
12190: 					ORDER BY `ticket`.`id_seat`
12191: 					) as sold_seats,
12192: 					`trip`.`id_schedule` as sc_id,
12193: 					`schedule`.`start_datetime` as sc_datetime,
12194: 					`schedule`.`only_date` as sc_only_date,
12195: 					`schedule`.`time_zone` as sc_time_zone,
12196: 					`schedule`.`id_price_time_function` as sc_price_time_function,
12197: 					`schedule`.`currency` as sc_currency,
12198: 					`schedule`.`currency_priority` as sc_currency_priority,
12199: 					`schedule`.`fee` as sc_fee,
12200: 					`schedule`.`tariff` as sc_tariff,
12201: 					`schedule`.`tariff_priority` as sc_tariff_priority
12202: 					";
12203: 				$sql_left_join_schedule = "LEFT JOIN `schedule` USING (`id_schedule`)";
12204: 				if (!empty($filter))
12205: 				{
12206: 					$filter = explode(',',$filter);
12207: 					foreach($filter as $key=>$value)
12208: 					{
12209: 						$filter[$key] = 'sc_id' . chr(0) . trim($value);
12210: 					}
12211: 					$filter = implode("','",$filter);
12212: 					$sql_where .= " AND `trip`.`from` in ('" . $filter . "')";
12213: 				}
12214: 			}
12215: 			else
12216: 			{
12217: 				if (!empty($filter))
12218: 				{
12219: 					if ($filter = 'sd_cd')
12220: 					{
12221: 						$sql_where .= " AND (`start_plan_datetime` < now() AND `complete_plan_datetime` > now())";
12222: 					}
12223: 				}
12224: 			}
12225: 			if (!empty($id_order) && $type !== 'processing')
12226: 			{
12227: 				$sql_where .= " AND `order_trip`.`id_order` in (" . $id_order . ")";
12228: 			}
12229: 			if ($this->id_role == 2 || $this->id_role == 4)
12230: 			{
12231: 				$orders = ",
12232: 					IF(`order_trip`.`id_order` IS NULL,NULL,
12233: 						GROUP_CONCAT(
12234: 							CONCAT_WS(0x00,
12235: 								`order_trip`.`id_order`,
12236: 								`order_trip`.`offer_order_datetime`,
12237: 								`order_trip`.`select_trip_datetime`
12238: 							)
12239: 						SEPARATOR 0x01) 
12240: 					) as orders";
12241: 				$sql_left_join = "LEFT JOIN `order_trip` USING (`id_trip`)";
12242: 				if ($this->id_role == 2) $sql_where .= " AND `trip`.`driver` = '" . $_SESSION[UID] . "'";
12243: 			}
12244: 			elseif ($this->id_role == 1 || $this->id_role == 5)
12245: 			{
12246: 				$sql_where .= " AND `trip`.`driver` != '" . $_SESSION[UID] . "'";
12247: 			}
12248: 			if ($type == 'active')
12249: 			{
12250: 				if ($this->id_role == 1 || $this->id_role == 5)
12251: 				{
12252: 					$orders = ",
12253: 						IF(`order_trip`.`id_order` IS NULL,NULL,
12254: 							GROUP_CONCAT(IF(`order`.`client` = '" . $_SESSION[UID] . "',
12255: 								CONCAT_WS(0x00,
12256: 									`order_trip`.`id_order`,
12257: 									`order_trip`.`offer_order_datetime`,
12258: 									`order_trip`.`select_trip_datetime`
12259: 								),
12260: 							NULL) SEPARATOR 0x01) 
12261: 						) as orders";
12262: 					$sql_left_join = "LEFT JOIN `order_trip` USING (`id_trip`)
12263: 					LEFT JOIN `order` ON `order`.`id_order` = `order_trip`.`id_order`";
12264: 					$sql_where .= " AND (
12265: 								SELECT
12266: 									`order_trip`.`id_trip`
12267: 								FROM
12268: 									`order_trip`
12269: 								LEFT JOIN `order` USING (`id_order`)
12270: 								WHERE
12271: 									`order_trip`.`id_trip` = `trip`.`id_trip` AND 
12272: 									`order`.`client` = '" . $_SESSION[UID] . "' AND 
12273: 									IF(`trip`.`looking_for_clients` = 1,1,IF(`order_trip`.`select_trip_datetime` = 0,0,1))
12274: 								LIMIT 1
12275: 							) IS NOT NULL";
12276: 				}
12277: 				$sql_where .= " AND `trip`.`complete_datetime` = 0";
12278: 			}
12279: 			elseif ($type == 'processing')
12280: 			{
12281: 				if ($this->id_role == 1 || $this->id_role == 5)
12282: 				{
12283: 					$sql_left_join = "LEFT JOIN (
12284: 								SELECT
12285: 									`order_trip`.`id_trip`
12286: 								FROM
12287: 									`order_trip`
12288: 								LEFT JOIN `order` USING (`id_order`)
12289: 								WHERE
12290: 									`order`.`client` = '" . $_SESSION[UID] . "'
12291: 								LIMIT 1
12292: 							) ot USING (`id_trip`)";
12293: 					$sql_where .= " AND `ot`.`id_trip` IS NULL AND `trip`.`looking_for_clients` = 1 AND `trip`.`complete_datetime` = 0";
12294: 				}
12295: 				else
12296: 				{
12297: 					return $this->showError('404', 'error', 'wrong role of user');
12298: 				}
12299: 			}
12300: 			else
12301: 			{
12302: 				if ($this->id_role == 1 || $this->id_role == 5)
12303: 				{
12304: 					$orders = ",
12305: 						IF(`order_trip`.`id_order` IS NULL,NULL,
12306: 							GROUP_CONCAT(IF(`order`.`client` = '" . $_SESSION[UID] . "',
12307: 								CONCAT_WS(0x00,
12308: 									`order_trip`.`id_order`,
12309: 									`order_trip`.`offer_order_datetime`,
12310: 									`order_trip`.`select_trip_datetime`
12311: 								),
12312: 							NULL) SEPARATOR 0x01) 
12313: 						) as orders";
12314: 					$sql_left_join = "LEFT JOIN `order_trip` USING (`id_trip`)
12315: 					LEFT JOIN `order` ON `order`.`id_order` = `order_trip`.`id_order`";
12316: 				}
12317: 			}
12318: 
12319: 			$s = "SELECT			
12320: 					`trip`.`id_trip` as t_id,
12321: 					`trip`.`driver` as u_id,
12322: 					`trip`.`from` as t_start_address,
12323: 					`trip`.`from_lat` as t_start_latitude,
12324: 					`trip`.`from_lng` as t_start_longitude,
12325: 					`trip`.`to` as t_destination_address,
12326: 					`trip`.`to_lat` as t_destination_latitude,
12327: 					`trip`.`to_lng` as t_destination_longitude,
12328: 					`trip`.`start_plan_datetime_interval` as t_start_datetime_interval,
12329: 					`trip`.`start_plan_datetime` as t_start_datetime,
12330: 					`trip`.`complete_plan_datetime` as t_complete_datetime,
12331: 					`trip`.`start_datetime` as t_start_real_datetime,
12332: 					`trip`.`complete_datetime` as t_complete_real_datetime,
12333: 					`trip`.`last_edit_datetime` as t_edit_datetime,
12334: 					`trip`.`last_edit_user` as e_u_id,
12335: 					`trip`.`create_datetime` as t_create_datetime,
12336: 					`trip`.`create_user` as c_u_id,
12337: 					`trip`.`json` as t_options,
12338: 					`trip`.`looking_for_clients` as t_looking_for_clients,
12339: 					`trip`.`canceled` as t_canceled,
12340: 					`trip`.`id_price_time_function` as price_time_function,
12341: 					`trip`.`currency`,
12342: 					`trip`.`currency_priority`,
12343: 					`trip`.`fee`,
12344: 					`trip`.`tariff`,
12345: 					`trip`.`tariff_priority`,
12346: 					`trip`.`id_aggregator` as ag_id
12347: 					" . $orders . "
12348: 					" . $cart_sold_seats . "
12349: 				FROM `trip`
12350: 				" . $sql_left_join . "
```

### `selectUser` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `639-862`
```php
639: 		public function selectUser($id_user = "", $users_props = array(), $field_types = array())
640: 		{
641: 			if (empty($_SESSION[UID])) {
642: 				$prop_visibility = 1;
643: 				return $this->showError('404', 'error', 'unauthorized access');
644: 			}
645: 			$sql_add = "
646: 					`phone` as u_phone,
647: 					`phone_is_verified` as u_phone_checked,
648: 					`email` as u_email,
649: 					`email_is_verified` as u_email_checked,
650: 					`birthday_date` as u_birthday,
651: 					`referral_code` as ref_code,
652: 					`referrer` as referrer_u_id,
653: 					`tg` as u_tg,
654: 					`tg_is_verified` as u_tg_checked,
655: 					`id_user_upper` as u_upper,
656: 					`wa` as u_wa,
657: 					`wa_is_verified` as u_wa_checked,";
658: 			$sql_limit = "";
659: 			if (empty($id_user))
660: 			{	
661: 				if ($this->id_role != 4)
662: 				{
663: 					$prop_visibility = 4;
664: 					$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
665: 					$sql_limit = "LIMIT 1";
666: 				}
667: 				else
668: 				{
669: 					$prop_visibility = 8;
670: 					$sql_where = "";
671: 					$sql_limit = "LIMIT " . $this->limit_offset . ", " . $this->limit_row_count;
672: 				}
673: 			}
674: 			else
675: 			{	
676: 				if ($id_user == 'authorized' || $id_user == $_SESSION[UID]) 
677: 				{
678: 					$prop_visibility = 4;
679: 					$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
680: 					$sql_limit = "LIMIT 1";
681: 				}
682: 				else
683: 				{
684: 					$prop_visibility = 8;
685: 					if ($this->id_role != 4) 
686: 					{
687: 						$sql_add = "";
688: 						$prop_visibility = 2;
689: 					}
690: 					$sql_where = "AND `id_user` in (" . $id_user . ")";
691: 					$sql_limit = "LIMIT " . $this->constant['limit_row_count_max'];
692: 				}
693: 			}
694: 
695: 			$sql_props = array();
696: 			$find_props = array();
697: 			foreach($users_props as $id_users_prop=>$prop_arr)
698: 			{
699: 				if ($prop_arr['visibility'] & $prop_visibility)
700: 				{
701: 					$value_type_str = $field_types[$prop_arr['value_type']]['var'];
702: 					$some_str = empty($prop_arr['some']) ? '' : '_some';
703: 					$table_name = "users_prop_items_$value_type_str$some_str";
704: 					$prop_name = $prop_arr['var'];
705: 					$sql_props[] = "(SELECT
706: 						GROUP_CONCAT(
707: 							IFNULL(`value`,0x02)
708: 						SEPARATOR 0x01)
709: 					 FROM `$table_name`
710: 					 WHERE `$table_name`.`id_user` = `users`.`id_user` AND `id_users_prop` = '$id_users_prop'
711: 					) as '$prop_name'";
712: 					$find_props[] = $prop_arr['var'];
713: 				}
714: 				elseif ($prop_visibility == 8 && ($prop_arr['visibility'] & 4))
715: 				{
716: 					$value_type_str = $field_types[$prop_arr['value_type']]['var'];
717: 					$some_str = empty($prop_arr['some']) ? '' : '_some';
718: 					$table_name = "users_prop_items_$value_type_str$some_str";
719: 					$prop_name = $prop_arr['var'];
720: 					$sql_props[] = "IF(`users`.`id_user` = '{$_SESSION[UID]}',(SELECT
721: 						GROUP_CONCAT(
722: 							IFNULL(`value`,0x02)
723: 						SEPARATOR 0x01)
724: 					 FROM `$table_name`
725: 					 WHERE `$table_name`.`id_user` = `users`.`id_user` AND `id_users_prop` = '$id_users_prop'
726: 					),NULL) as '$prop_name'";
727: 					$find_props[] = $prop_arr['var'];
728: 				}
729: 			}
730: 			if (empty($sql_props))
731: 			{
732: 				$sql_props = '';
733: 			}
734: 			else
735: 			{
736: 				$sql_props = implode(',',$sql_props);
737: 				$sql_props = ",$sql_props";
738: 			}
739: 
740: 			$s = "SELECT 
741: 					`id_user` as u_id,
742: 					`id_role` as u_role,
743: 					`name` as u_name,
744: 					`family` as u_family,
745: 					`middle` as u_middle,
746: 					" . $sql_add . "
747: 					`photo_link` as u_photo,
748: 					`id_lang` as u_lang,
749: 					`currency` as u_currency,
750: 					`id_city` as u_city,
751: 					`tips` as u_tips,
752: 					`language_skills` as u_lang_skills,
753: 					`description` as u_description,
754: 					`id_navigation` as u_gps_software,
755: 					`id_verification_status` as u_check_state,
756: 					`active` as u_active,			
757: 					`out_order` as out_drive,
758: 					`out_order_to` as out_address,
759: 					`out_order_to_lat` as out_latitude,
760: 					`out_order_to_lng` as out_longitude,
761: 					`out_order_complete_datetime` as out_est_datetime,
762: 					`out_order_from` as out_s_address,
763: 					`out_order_from_lat` as out_s_latitude,
764: 					`out_order_from_lng` as out_s_longitude,
765: 					`out_order_passengers_count` as out_passengers,
766: 					`out_order_luggage_count` as out_luggage,
767: 					`json` as u_details,
768: 					(SELECT
769: 						GROUP_CONCAT(`users_order_comment`.`id_order_comment` SEPARATOR ',')
770: 					 FROM `users_order_comment`
771: 					 WHERE `users_order_comment`.`id_user` = `users`.`id_user`
772: 					) as b_comments,
773: 					(SELECT
774: 						GROUP_CONCAT(`users_service`.`id_service` SEPARATOR ',')
775: 					 FROM `users_service`
776: 					 WHERE `users_service`.`id_user` = `users`.`id_user`
777: 					) as b_services,
778: 					(SELECT
779: 						GROUP_CONCAT(
780: 							CONCAT_WS('|',`users_order_location`.`id_order_location`,`users_order_location`.`basic`) 
781: 						SEPARATOR ';')
782: 					 FROM `users_order_location`
783: 					 WHERE `users_order_location`.`id_user` = `users`.`id_user`
784: 					) as b_location_classes,
785: 					`id_schedule` as sc_id$sql_props
786: 				FROM `users`
787: 				WHERE
788: 					`deleted` = '0'
789: 				" . $sql_where . "
790: 				" . $sql_limit . "
791: 				";
792: 
793: 			$q = query($s);
794: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
795: 			
796: 			$out = array('user' => array());
797: 			while ($d = fetch_assoc($q))
798: 			{
799: 				$d['u_ban'] = get_user_ban_status($d['u_id']);
800: 				$d['u_photo'] = $d['u_photo'] ? url($d['u_photo'],FILE_PATH) : '';
801: 				add_time_zone($d['out_est_datetime']);
802: 				if (!empty($d['b_comments'])) $d['b_comments'] = explode(',',$d['b_comments']);		
803: 				if (!empty($d['b_services'])) $d['b_services'] = explode(',',$d['b_services']);			
804: 				if (!empty($d['b_location_classes']))
805: 				{
806: 					$d['b_location_classes'] = explode(';',$d['b_location_classes']);
807: 					foreach ($d['b_location_classes'] as $key=>$value)
808: 					{
809: 						$d['b_location_classes'][$key] = array();
810: 						list(
811: 							$d['b_location_classes'][$key]['b_location_class'],
812: 							$d['b_location_classes'][$key]['basic']
813: 							)= explode('|',$value);
814: 						$d['b_location_classes'][$key]['basic'] = (int)$d['b_location_classes'][$key]['basic'];
815: 					}
816: 				}
817: 				if (isset($d['u_phone_checked'])) $d['u_phone_checked'] = (int)$d['u_phone_checked'];
818: 				$d['u_active'] = (int)$d['u_active'];
819: 				$d['out_drive'] = (int)$d['out_drive'];	
820: 				$d['u_details'] = json_decode($d['u_details'],true);
821: 	
822: 				foreach($find_props as $find_prop)
823: 				{
824: 					if ($d[$find_prop] !== NULL)
825: 					{
826: 						$d[$find_prop] = explode(chr(1),$d[$find_prop]);
827: 						foreach($d[$find_prop] as $p_key=>$p_val)
828: 						{
829: 							if ($p_val === chr(2)) $d[$find_prop][$p_key] = NULL;
830: 						}
831: 						$d['props'][$find_prop] = $d[$find_prop];
832: 					}
833: 					unset($d[$find_prop]);
834: 				}
835: 
836: 				$out['user'][$d['u_id']] = $d;
837: 			}
838: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
839: 			return array(
840: 				'code' 		=>	'200',
841: 				'status' 	=>	'success',		
842: 				'data' 		=>	$out,
843: 				'auth_user' =>	array(
844: 									'u_id' => $_SESSION[UID],
845: 									'u_name' => $_SESSION['name'],
846: 									'u_family' => $_SESSION['family'],
847: 									'u_middle' => $_SESSION['middle'],
848: 									'u_email' => $_SESSION['email'],
849: 									'u_phone' => $_SESSION['phone'],
850: 									'u_role' => $_SESSION['id_role'],
851: 									'u_a_role' => $this->id_role,
852: 									'u_check_state' => $_SESSION['id_verification_status'],
853: 									'u_ban' => $_SESSION['user_ban_status'],
854: 									'u_active' => $_SESSION['active'],
855: 									'u_photo' => $_SESSION['photo_link'],
856: 									'u_birthday' => $_SESSION['birthday_date'],
857: 									'u_lang' => $_SESSION['id_lang'],
858: 									'u_currency' => $_SESSION['currency'],
859: 									'u_gps_software' => $_SESSION['id_navigation']
```

### `sendMessage` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `21709-22150`
```php
21709: 		public function sendMessage($name = '', $value = '', $id_message_upper = NULL, $id_message_type = NULL, $from_to = '', $co_ids = '', $co_id = NULL, $code = NULL, $owner_types = array(), $roles = array())
21710: 		{	
21711: 			if (empty($_SESSION[UID])) {
21712: 				return $this->showError('404', 'error', 'unauthorized access');
21713: 			}
21714: 			
21715: 			if (empty($id_message_type)) $id_message_type = 1;
21716: 			$id_user = $_SESSION[UID];
21717: 			if ($code !== NULL)
21718: 			{
21719: 				if (empty($co_id)) return $this->showError('404', 'error', 'empty co_id');
21720: 				$from_to = NULL;
21721: 				$co_ids = NULL;
21722: 			}
21723: 			elseif ($id_message_type == 2 || $id_message_type == 5)
21724: 			{
21725: 				if (empty($co_ids)) return $this->showError('404', 'error', 'empty co_ids');
21726: 				$co_id = NULL;
21727: 			}
21728: 			elseif ($id_message_type == 3 || $id_message_type == 4)
21729: 			{
21730: 				$co_id = NULL;
21731: 				$co_ids = NULL;
21732: 			}
21733: 			elseif ($id_message_type == 6)
21734: 			{
21735: 				if (empty($co_id)) return $this->showError('404', 'error', 'empty co_id for message');
21736: 				$co_ids = NULL;
21737: 			}
21738: 			else
21739: 			{
21740: 				$co_id = NULL;
21741: 			}
21742: 
21743: 			if ($code === NULL && empty($from_to))
21744: 			{
21745: 				return $this->showError('404', 'error', 'empty from_to');
21746: 			}
21747: 
21748: 			if (!empty($from_to))
21749: 			{
21750: 				$from_to = $this->parseOwnerFromTo($from_to,$owner_types);
21751: 				if (!empty($from_to['error'])) return $this->showError('404', 'error', $from_to['error']);
21752: 				list($from_owner,$from_o_type,$to_owner,$to_o_type,$owner_types_filtered) = $from_to;
21753: 		
21754: 				if ($id_message_type == 2 || $id_message_type == 5 || $id_message_type == 3 || $id_message_type == 4)
21755: 				{
21756: 					if ($from_o_type != 1 || $to_o_type != 1)
21757: 					{
21758: 						return $this->showError('404', 'error', 'forbidden m_type');
21759: 					}
21760: 				}
21761: 			}
21762: 			
21763: 			$co_id_list = array();
21764: 			if (!empty($co_id))
21765: 			{
21766: 				$co_id_list[$co_id] = true;
21767: 			}
21768: 			if (!empty($co_ids))
21769: 			{
21770: 				$co_ids = json_decode($co_ids,true);
21771: 				
21772: 				if (empty($co_ids) || !is_array($co_ids)) 
21773: 				{
21774: 					return $this->showError('404', 'error', 'wrong co_ids');
21775: 				}
21776: 				foreach($co_ids as $val_arr)
21777: 				{
21778: 					if (!is_array($val_arr) || !isset($val_arr[0]) || !isset($val_arr[1])) return $this->showError('404', 'error', 'wrong co_ids element');
21779: 					if ($val_arr[0] != (int)$val_arr[0] || $val_arr[1] != (int)$val_arr[1]) return $this->showError('404', 'error', 'wrong co_ids element number');
21780: 					$co_id_list[$val_arr[0]] = true;
21781: 					$co_id_list[$val_arr[1]] = true;
21782: 				}
21783: 			}
21784: 			if (!empty($co_id_list))
21785: 			{
21786: 				$where_co_id = implode(',',array_keys($co_id_list));
21787: 				$s = "SELECT 
21788: 						`id_contact_item`,
21789: 						`owner`,
21790: 						`id_owner_type`,
21791: 						`id_contact_type`,
21792: 						`contact_number`,
21793: 						`contact_id`,
21794: 						`contact_link`
21795: 					FROM `contact_items`
21796: 					WHERE
21797: 						`id_contact_item` in ($where_co_id)
21798: 					";
21799: 
21800: 				$q = query($s);
21801: 				if ($q === false) return $this->showError('404', 'error', 'co_id: select failed');
21802: 				
21803: 				$contacts = array();
21804: 				while ($d = fetch_assoc($q))
21805: 				{
21806: 					$contacts[$d['id_contact_item']] = $d;
21807: 				}
21808: 			}
21809: 			if (!empty($co_ids))
21810: 			{
21811: 				foreach($co_ids as $val_arr)
21812: 				{
21813: 					if (empty($contacts[$val_arr[0]]) || empty($contacts[$val_arr[1]])) return $this->showError('404', 'error', 'co_ids element not found');
21814: 					$co_id_from = $contacts[$val_arr[0]];
21815: 					$co_id_to = $contacts[$val_arr[1]];
21816: 					if ($co_id_from['owner'] != $from_owner || $co_id_from['id_owner_type'] != $from_o_type
21817: 						|| $co_id_to['owner'] != $to_owner || $co_id_to['id_owner_type'] != $to_o_type) return $this->showError('404', 'error', 'contact does not belong to sender or recipient');
21818: 					if ($co_id_from['id_contact_type'] != $co_id_to['id_contact_type']) return $this->showError('404', 'error', 'different types of contacts');
21819: 
21820: 					if ($co_id_from['id_contact_type'] != 7) return $this->showError('404', 'error', 'types of contact does not support sending');
21821: 					if (empty($co_id_from['contact_number'])) return $this->showError('404', 'error', 'sender contact without number');
21822: 					if (empty($co_id_to['contact_link'])) return $this->showError('404', 'error', 'recipient contact without requisite');
21823: 				}
21824: 			}
21825: 			if (!empty($co_id))
21826: 			{
21827: 				if (empty($contacts[$co_id])) return $this->showError('404', 'error', 'co_id not found');
21828: 				if ($code === NULL)
21829: 				{
21830: 					if ($contacts[$co_id]['owner'] != $from_owner || $contacts[$co_id]['id_owner_type'] != $from_o_type) return $this->showError('404', 'error', 'contact does not belong to sender');
21831: 				}
21832: 				else
21833: 				{
21834: 					if ($contacts[$co_id]['id_contact_type'] != 7) return $this->showError('404', 'error', 'code: types of contact does not support sending');
21835: 					if (empty($contacts[$co_id]['contact_number'])) return $this->showError('404', 'error', 'code: sender contact without number');
21836: 					$from_owner = $contacts[$co_id]['owner'];
21837: 					$from_o_type = $contacts[$co_id]['id_owner_type'];
21838: 					if ($this->id_role != 4 && $from_o_type != 1) return $this->showError('404', 'error', 'code: not enough rights');
21839: 					$owner_types_filtered = array($from_o_type => $owner_types[$from_o_type]);
21840: 				}
21841: 			}
21842: 
21843: 			list($error,$available_owner,$sql_where_owner,$sql_where_from,$sql_where_to) = array_pad($this->parseOwnerAvailable($id_user,$owner_types_filtered),5,'');
21844: 			if (!empty($error)) return $this->showError('404', 'error', $error);
21845: 			if (empty($available_owner[$from_o_type][$from_owner])) return $this->showError('404', 'error', 'forbidden from_owner');
21846: 
21847: 			if ($code !== NULL)
21848: 			{
21849: 				$response = send_msg_to_tg('','', $contacts[$co_id]['contact_number'],'',$code);
21850: 				if (!empty($response['error'])) return $this->showError('404', 'error', array('code send error'=>$response['error']));
21851: 				if ($response == 'code sent') return $this->showError('404', 'error', 'new code sent');
21852: 				return array('code'=>'200','status'=>'success');
21853: 			}
21854: 
21855: 			$to_owner_exist = $this->parseOwnerExists($to_o_type,$to_owner,$owner_types,$roles);
21856: 			if (!empty($to_owner_exist['error'])) return $this->showError('404', 'error', $to_owner_exist['error']);
21857: 			if (empty($to_owner_exist)) return $this->showError('404', 'error', 'to_owner not found');
21858: 
21859: 			$out = array();
21860: 			
21861: 			if ($id_message_type == 1 || $id_message_type == 6 || $id_message_type == 2 || $id_message_type == 5)
21862: 			{
21863: 				if ($from_o_type == 1 && $to_o_type == 1)
21864: 				{
21865: 					$s = "SELECT 
21866: 							(SELECT 
21867: 								IF(`message`.`id_message_type` = 3, 1, 0)
21868: 							FROM `message`
21869: 							WHERE
21870: 								`message`.`id_message_type` in (3,4) AND `message`.`sender_owner_type`= '$from_o_type' AND `message`.`sender_owner`= '$from_owner' AND `message`.`recipient_owner_type`= '$to_o_type' AND `message`.`recipient_owner`= '$to_owner' AND `message`.`active_status` = 1
21871: 							) as sender,
21872: 							(SELECT
21873: 								IF(`message`.`id_message_type` = 3, 1, 0) as status
21874: 							FROM `message`
21875: 							WHERE
21876: 								`message`.`id_message_type` in (3,4) AND `message`.`sender_owner_type`= '$to_o_type' AND `message`.`sender_owner`= '$to_owner' AND `message`.`recipient_owner_type`= '$from_o_type' AND `message`.`recipient_owner`= '$from_owner' AND `message`.`active_status` = 1
21877: 							) as recipient
21878: 						";
21879: 
21880: 					$q = query($s);
21881: 					if ($q === false) return $this->showError('404', 'error', 'select status failed');
21882: 					
21883: 					$msg = array();
21884: 					$d = fetch_assoc($q);
21885: 					foreach(array('sender','recipient') as $participant)
21886: 					{
21887: 						if (isset($d[$participant]))
21888: 						{
21889: 							if (empty($d[$participant]))
21890: 							{
21891: 								$msg[] = "$participant blocked correspondence";
21892: 							}
21893: 						}
21894: 						else
21895: 						{
21896: 							$msg[] = "$participant must open a correspondence";
21897: 						}
21898: 					}
21899: 					if (!empty($msg)) return $this->showError('404', 'error', $msg);
21900: 
21901: 					if (!empty($co_ids))
21902: 					{
21903: 						if ($id_message_type == 1)
21904: 						{
21905: 							$s = array();
21906: 							foreach($co_ids as $val_arr)
21907: 							{
21908: 								$co_id_from = $val_arr[0];
21909: 								$co_id_to = $val_arr[1];
21910: 								
21911: 								$s[] = "SELECT 
21912: 										(SELECT 
21913: 											IF(`message`.`id_message_type` = 2, 1, 0)
21914: 										FROM `message`
21915: 										WHERE
21916: 											`message`.`id_message_type` in (2,5) AND `message`.`sender_contact_item`= '$co_id_from' AND `message`.`recipient_contact_item`= '$co_id_to' AND `message`.`active_status` = 1
21917: 										) as sender,
21918: 										(SELECT
21919: 											IF(`message`.`id_message_type` = 2, 1, 0)
21920: 										FROM `message`
21921: 										WHERE
21922: 											`message`.`id_message_type` in (2,5) AND `message`.`sender_contact_item`= '$co_id_to' AND `message`.`recipient_contact_item`= '$co_id_from' AND `message`.`active_status` = 1
21923: 										) as recipient
21924: 									";
21925: 							}
21926: 							$s = implode(' UNION ', $s);
21927: 
21928: 							$q = query($s);
21929: 							if ($q === false) return $this->showError('404', 'error', 'union select status failed');
```

### `sendToTicketBuyerEmail` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `19007-19550`
```php
19007: 		public function sendToTicketBuyerEmail($resend = false, $id_trip = "", $seat = "", $subject = "", $body = "", $file = '', $id_user = "", $u_name = NULL, $email = "", $ignore_order = false, $price = NULL, $lang_vls = array(), $price_time_functions = array(), $s_data_stt = array(), $aggregators = array())
19008: 		{
19009: 			if (empty($_SESSION[UID])) {
19010: 				return $this->showError('404', 'error', 'unauthorized access');
19011: 			}
19012: 
19013: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
19014: 
19015: 			if (empty($seat)) return $this->showError('404', 'error', 'empty seat');
19016: 
19017: 			if ($ignore_order === false)
19018: 			{
19019: 				$id_user = '';
19020: 				$price = NULL;
19021: 			}
19022: 
19023: 			if (!empty($file))
19024: 			{
19025: 				@$file = json_decode($file,true);				
19026: 				if (empty($file) || !is_array($file)) 
19027: 				{
19028: 					return $this->showError('404', 'error', 'wrong file data');
19029: 				}
19030: 			}
19031: 
19032: 			$extended = false;
19033: 			if ($subject == 'template' || $body == 'template')
19034: 			{
19035: 				$extended = true;
19036: 			}
19037: 			elseif (!empty($file))
19038: 			{
19039: 				foreach ($file as $f)
19040: 				{
19041: 					if (isset($f['base64']))
19042: 					{
19043: 						if ($f['base64'] == 'pdf')
19044: 						{
19045: 							$extended = true;
19046: 							break;
19047: 						}
19048: 					}
19049: 				}
19050: 			}
19051: 
19052: 			$sql_field = $sql_left_join = '';
19053: 			if ($extended == true)
19054: 			{
19055: 				$sql_field = ",
19056: 					`order`.`options`,
19057: 					`order`.`sum`,
19058: 					`order`.`currency` as sum_currency,
19059: 					`ticket`.`number`,
19060: 					`ticket`.`code`,
19061: 					`ticket`.`code_qr_base64`,
19062: 					`ticket`.`tariff` as ti_tariff,
19063: 					`ticket`.`currency` as ti_currency,
19064: 					`schedule`.`id_schedule`,
19065: 					`schedule`.`team1`,
19066: 					`schedule`.`team2`,
19067: 					`schedule`.`id_stadium` as stadium,
19068: 					`schedule`.`id_tournament` as tournament,
19069: 					`schedule`.`start_datetime` as datetime,
19070: 					`schedule`.`only_date`,
19071: 					`schedule`.`time_zone`,
19072: 					`schedule`.`code_ean_base64`,						
19073: 					`schedule`.`id_price_time_function` as sc_price_time_function,
19074: 					`schedule`.`currency` as sc_currency,
19075: 					`schedule`.`currency_priority` as sc_currency_priority,
19076: 					`schedule`.`fee` as sc_fee,
19077: 					`schedule`.`tariff` as sc_tariff,
19078: 					`schedule`.`tariff_priority` as sc_tariff_priority,						
19079: 					`trip`.`id_price_time_function` as price_time_function,
19080: 					`trip`.`currency`,
19081: 					`trip`.`currency_priority`,
19082: 					`trip`.`fee`,
19083: 					`trip`.`tariff`,
19084: 					`trip`.`tariff_priority`,
19085: 					`trip`.`create_datetime`,
19086: 					`trip`.`id_aggregator` as ag_id,
19087: 					`users`.`name`,
19088: 					`users`.`family`,
19089: 					`users`.`middle`";
19090: 				$sql_left_join = "LEFT JOIN `schedule` ON `schedule`.`id_schedule` = `trip`.`id_schedule`";
19091: 			}
19092: 
19093: 			$s = "SELECT			
19094: 					`trip`.`id_trip`,
19095: 					`trip`.`driver`,
19096: 					`trip`.`json`,
19097: 					`ticket`.`blob_link`,
19098: 					`ticket`.`blob_mime`,
19099: 					`ticket`.`blob_ext_w._dot`,
19100: 					`ticket`.`id_seat`,				
19101: 					`ticket`.`id_trip_seat`,
19102: 					`ticket`.`id_order`,
19103: 					`order`.`client`,
19104: 					`users`.`email`,
19105: 					`order`.`pay_datetime`$sql_field
19106: 				FROM `trip`
19107: 				LEFT JOIN `ticket` ON `ticket`.`id_trip` = `trip`.`id_trip` AND `ticket`.`id_seat` = '" . $seat . "'
19108: 				LEFT JOIN `order` ON `order`.`id_order` = `ticket`.`id_order`
19109: 				LEFT JOIN `users` ON `users`.`id_user` = `order`.`client`
19110: 				$sql_left_join
19111: 				WHERE	
19112: 					`trip`.`id_trip` = '" . $id_trip . "'
19113: 				LIMIT 1
19114: 				";
19115: 			
19116: 			$q = query($s);
19117: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
19118: 
19119: 			$d = fetch_assoc($q);
19120: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
19121: 
19122: 			$t_options = json_decode($d['json'],true);
19123: 			if (empty($t_options['seats_sold']))
19124: 			{
19125: 				if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
19126: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller');
19127: 				if ($ignore_order === false && empty($d['client'])) return $this->showError('404', 'error', 'buyer not found');				
19128: 				if ($ignore_order === false && $d['pay_datetime'] == '0000-00-00 00:00:00') return $this->showError('404', 'error', 'unpaid order');
19129: 				if (empty($email) && empty($id_user))
19130: 				{
19131: 					if (empty($d['email'])) return $this->showError('404', 'error', 'empty buyer email');
19132: 					$email = $d['email'];
19133: 				}
19134: 			}
19135: 			else
19136: 			{
19137: 				if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
19138: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller of trip');
19139: 				if (!array($t_options['seats_sold'][$seat]) || count($t_options['seats_sold'][$seat]) < 2) 
19140: 				{
19141: 					if ($ignore_order === false) return $this->showError('404', 'error', 'seat buyer not found');
19142: 				}
19143: 				else
19144: 				{
19145: 					$buyer = $t_options['seats_sold'][$seat][1];
19146: 
19147: 					$s = "SELECT 
19148: 							`id_user`,
19149: 							`email`,
19150: 							`users`.`name`,
19151: 							`users`.`family`,
19152: 							`users`.`middle`
19153: 						FROM `users`
19154: 						WHERE
19155: 							`id_user` = '" . $buyer . "'
19156: 						";
19157: 
19158: 					$q = query($s);
19159: 					if ($q === false) return $this->showError('404', 'error', 'user select failed');
19160: 
19161: 					$d_u = fetch_assoc($q);
19162: 					if (empty($d_u['id_user'])) return $this->showError('404', 'error', 'buyer user not found');
19163: 					if (empty($email) && empty($id_user))
19164: 					{
19165: 						if (empty($d_u['email'])) return $this->showError('404', 'error', 'buyer with empty email');
19166: 						$email = $d_u['email'];
19167: 					}
19168: 				}
19169: 			}
19170: 
19171: 			if ($resend === true)
19172: 			{
19173: 				if (empty($d['client'])) return $this->showError('404', 'error', 'ticket without buyer');	
19174: 				$api_use = true;
19175: 				$order = $d['id_order'];
19176: 				$status = 'succeeded';
19177: 				$res = require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'controllers/c_stripe.php');
19178: 				if (!empty($res['error'])) return $this->showError('404', 'error', $res['error']);
19179: 				return array(
19180: 					'code' 		=>	'200',
19181: 					'status' 	=>	'success'
19182: 				);
19183: 			}
19184: 
19185: 			if (!empty($id_user))
19186: 			{
19187: 				$s = "SELECT 
19188: 						`id_user`,
19189: 						`email`,
19190: 						`users`.`name`,
19191: 						`users`.`family`,
19192: 						`users`.`middle`
19193: 					FROM `users`
19194: 					WHERE
19195: 						`id_user` = '" . $id_user . "'
19196: 					";
19197: 
19198: 				$q = query($s);
19199: 				if ($q === false) return $this->showError('404', 'error', 'select of user failed');
19200: 
19201: 				$d_u = fetch_assoc($q);
19202: 				if (empty($d_u['id_user'])) return $this->showError('404', 'error', 'user not found');
19203: 				if (empty($email))
19204: 				{
19205: 					if (empty($d_u['email'])) return $this->showError('404', 'error', 'user with empty email');
19206: 					$email = $d_u['email'];
19207: 				}
19208: 			}
19209: 
19210: 			$ticket_file_name = false;
19211: 			if ($d['blob_link'] === '')
19212: 			{
19213: 				$ticket_file = false;
19214: 			}
19215: 			else
19216: 			{
19217: 				$folder = $_SERVER['DOCUMENT_ROOT'] . CONFIG_USER_FILE_PATH . "trips/$id_trip/ticket/";
19218: 				$name = $d['blob_link'] !== NULL ? $d['blob_link'] : $d['id_trip_seat'];
19219: 				$path = "$folder$name";
19220: 				if (!file_exists($path)) 
19221: 				{
19222: 					$ticket_file = false;
19223: 				}
19224: 				else
19225: 				{
19226: 					$ticket_file = file_get_contents($path);
19227: 					$ext = '';
```

### `setCarIsArrived` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver`
- source range: `6799-6894`
```php
6799: 		public function setCarIsArrived($id_order = "")
6800: 		{
6801: 			if (empty($_SESSION[UID])) {
6802: 				return $this->showError('404', 'error', 'unauthorized access');
6803: 			}
6804: 			if ($this->id_role != 2)
6805: 			{
6806: 				return $this->showError('404', 'error', 'wrong user role');
6807: 			}
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
6831: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
6832: 
6833: 			$d = fetch_assoc($q);
6834: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6835: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 
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
6848: 			elseif ($d['id_order_driver_status'] == 2 )
6849: 			{
6850: 				return $this->showError('404', 'error', 'canceled performer');
6851: 			}
6852: 			elseif ($d['id_order_driver_status'] == 4 )
6853: 			{
6854: 				return $this->showError('404', 'error', 'arrive state has already been changed');
6855: 			}
6856: 			elseif ($d['id_order_driver_status'] == 5)
6857: 			{
6858: 				return $this->showError('404', 'error', 'started ride');
6859: 			}
6860: 			elseif ($d['id_order_driver_status'] == 6)
6861: 			{
6862: 				return $this->showError('404', 'error', 'completed booking');
6863: 			}
6864: 			
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
6877: 
6878: 			query($s);
6879: 	
6880: 			global $link;
6881: 			if (mysqli_affected_rows($link) === -1) 
6882: 			{
6883: 				return $this->showError('404', 'error', 'database update failed');
6884: 			}
6885: 			elseif (mysqli_affected_rows($link) === 0) 
6886: 			{
6887: 				return $this->showError('404', 'error', 'modified data not found');
6888: 			}
6889: 
6890: 			return array(
6891: 				'code' 		=>	'200',
6892: 				'status' 	=>	'success'
6893: 			);
6894: 		}
```

### `setCarUsed` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver`
- source range: `8967-9058`
```php
8967: 		public function setCarUsed($id_car = "")
8968: 		{
8969: 			if (empty($_SESSION[UID])) {
8970: 				return $this->showError('404', 'error', 'unauthorized access');
8971: 			}
8972: 			if ($this->id_role != 2)
8973: 			{
8974: 				return $this->showError('404', 'error', 'wrong user role');
8975: 			}
8976: 			if ($_SESSION['id_verification_status'] != 2)
8977: 			{
8978: 				return $this->showError('404', 'error', 'wrong user check state');
8979: 			}
8980: 
8981: 			$q = query("BEGIN");
8982: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
8983: 			
8984: 			$s = "SELECT
8985: 					`car`.`id_car`,
8986: 					GROUP_CONCAT(IF(`car_users`.`id_user` = '" . $_SESSION[UID] . "',`car_users`.`id_user`,NULL) SEPARATOR ',') as u_id,
8987: 					GROUP_CONCAT(IF(`car_users`.`used` = '1',`car_users`.`id_user`,NULL) SEPARATOR ',') as u_d_id
8988: 				FROM `car`
8989: 				LEFT JOIN `car_users` USING (`id_car`)
8990: 				WHERE
8991: 					`car`.`id_car` = '" . $id_car . "'
8992: 				GROUP BY
8993: 					`car_users`.`id_car`
8994: 				";
8995: 
8996: 			$q = query($s);
8997: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8998: 
8999: 			$d = fetch_assoc($q);
9000: 			if (empty($d['id_car'])) 
9001: 			{
9002: 				return $this->showError('404', 'error', 'car not found');
9003: 			}
9004: 			if (empty($d['u_id'])) 
9005: 			{
9006: 				return $this->showError('404', 'error', 'car is not this driver');
9007: 			}
9008: 			if ($d['u_d_id'] === $_SESSION[UID]) 
9009: 			{
9010: 				return $this->showError('404', 'error', 'car is already driven by this user');
9011: 			}
9012: 			elseif (!empty($d['u_d_id']))
9013: 			{
9014: 				return $this->showError('404', 'error', 'car is already driven');
9015: 			}
9016: 
9017: 			global $link;
9018: 
9019: 			$s = "UPDATE `car_users`
9020: 				SET 
9021: 					`used` = NULL
9022: 				WHERE
9023: 					`id_user` = '" . $_SESSION[UID] . "' AND `id_car` != '" . $id_car . "' AND `used` = '1'
9024: 				";
9025: 
9026: 			query($s);
9027: 
9028: 			if (mysqli_affected_rows($link) === -1) 
9029: 			{
9030: 				return $this->showError('404', 'error', 'update of database failed');
9031: 			}
9032: 			
9033: 			$s = "UPDATE `car_users`
9034: 				SET 
9035: 					`used` = '1'
9036: 				WHERE
9037: 					`id_car` = '" . $id_car . "' AND `id_user` = '" . $_SESSION[UID] . "'
9038: 				";
9039: 
9040: 			query($s);
9041: 			
9042: 			if (mysqli_affected_rows($link) === -1) 
9043: 			{
9044: 				return $this->showError('404', 'error', 'database update failed');
9045: 			}
9046: 			elseif (mysqli_affected_rows($link) === 0) 
9047: 			{
9048: 				return $this->showError('404', 'error', 'modified data not found');
9049: 			}
9050: 
9051: 			$q = query("COMMIT");
9052: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
9053: 
9054: 			return array(
9055: 				'code' 		=>	'200',
9056: 				'status' 	=>	'success'
9057: 			);
9058: 		}
```

### `setDriver` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `6180-6797`
```php
6180: 		public function setDriver($data = '', $id_order = "", $id_user = "", $appointed_performer = 0, $code = NULL, $trips = "")
6181: 		{
6182: 			if (empty($_SESSION[UID])) {
6183: 				return $this->showError('404', 'error', 'unauthorized access');
6184: 			}
6185: 
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
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
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
6209: 				$q = query($s);
6210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6211: 				
6212: 				$d = fetch_assoc($q);
6213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6214: 				if ($d['id_order_status'] != 1) return $this->showError('404', 'error', 'wrong booking state');
6215: 
6216: 				if ($d['client'] != $_SESSION[UID]) 
6217: 				{
6218: 					return $this->showError('404', 'error', $_SESSION[UID] . ' is not author');
6219: 				}
6220: 				if (empty($d['u_id'])) 
6221: 				{
6222: 					return $this->showError('404', 'error', $id_user . ' is not performer');
6223: 				}
6224: 				if (empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking not confirmed');
6225: 				if ($d['c_state'] != 1) 
6226: 				{
6227: 					return $this->showError('404', 'error', 'wrong booking driver state');
6228: 				}
6229: 
6230: 				$q = query("BEGIN");
6231: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
6232: 	
6233: 				$s = "UPDATE `order`,`order_driver`,(
6234: 							SELECT
6235: 
6236: 								'" . $id_order . "' as `id_order`,
6237: 								@c_c_count := COUNT(`id_order`) as current_car_count
6238: 							FROM
6239: 								`order_driver`
6240: 							WHERE
6241: 								`id_order` = '" . $id_order . "' AND 
6242: 								`id_order_driver_status` in (3,4,5,6)
6243: 
6244: 						) od
6245: 					SET 
6246: 						`order`.`id_order_status` = (SELECT @id := 
6247: 							IF(od.`current_car_count` < `order`.`car_count`-1,1,2)),
6248: 						`order`.`approve_datetime` = 
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
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
6263: 					";
6264: 				
6265: 				query($s);
6266: 
6267: 				global $link;
6268: 				$rows = mysqli_affected_rows($link);				
6269: 				if ($rows === -1) 
6270: 				{
6271: 					return $this->showError('404', 'error', 'database update failed');
6272: 				}
6273: 				elseif ($rows === 0) 
6274: 				{
6275: 					return $this->showError('404', 'error', 'modified data not found');
6276: 				}
6277: 				
6278: 				$s = "SELECT @c_c_count,@c_count,@id";
6279: 
6280: 				$q = query($s);
6281: 				if ($q === false) return $this->showError('404', 'error', 'variable select failed');
6282: 				$d_u = fetch_assoc($q);
6283: 
6284: 				$s = "UPDATE `order`
6285: 					SET 
6286: 						`last_edit_datetime` = now(),
6287: 						`last_edit_user` = '" .  $_SESSION[UID] . "',
6288: 						`price_estimate` = IF(`id_car_class` IS NULL,
6289: 							(SELECT 
6290: 								SUM(`price_estimate`)
6291: 							 FROM `order_driver`
6292: 							 WHERE
6293: 								`id_order` = '" . $id_order . "' AND 
6294: 								`id_order_driver_status` in (3,4,5,6)
6295: 							),`price_estimate`)
6296: 					WHERE `id_order` = '" . $id_order . "'
6297: 					";
6298: 
6299: 				$q = query($s);
6300: 				if ($q === false) return $this->showError('404', 'error', 'database timestamp update failed');
6301: 				
6302: 				if (!empty($trips))
6303: 				{
6304: 					$s = "SELECT
6305: 							COUNT(`id_trip`) as trips_count
6306: 						FROM `trip` 		
6307: 						WHERE	
6308: 							`id_trip` in (" . $trips . ") AND `driver` = '" . $id_user . "'
6309: 						";
6310: 
6311: 					$q = query($s);
6312: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
6313: 
6314: 					$d = fetch_assoc($q);
6315: 					$trips = explode(',', $trips);
6316: 					if ($d['trips_count'] != count($trips)) return $this->showError('404', 'error', 'driver is not trip author');
6317: 					
6318: 					$s = array();
6319: 					foreach ($trips as $id_trip)
6320: 					{
6321: 						$s[] = "('" . $id_order . "', '" . $id_trip . "', now(), now())";
6322: 					}
6323: 					$s = "INSERT INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`, `select_trip_datetime`) VALUES " . implode(",", $s) . "ON DUPLICATE KEY UPDATE `select_trip_datetime` = IF(`select_trip_datetime` = 0,now(),`select_trip_datetime`)";
6324: 
6325: 					$q = query($s);
6326: 					if ($q === false) return $this->showError('404', 'error', 'insert in database failed');
6327: 				}		
6328: 
6329: 				$q = query("COMMIT");
6330: 				if ($q === false) return $this->showError('404', 'error', 'commit query failed');
6331: 
6332: 				if ($rows === 1 && $d_u['id'] == 2) 
6333: 				{
6334: 					return $this->showError('404', 'error', 'only booking state modified');	
6335: 				}
6336: 				
6337: 				$out = array('current_cars_count' => (int)$d_u['@c_c_count']+1,'b_cars_count' => $d_u['@c_count']);
6338: 				if ($d_u['@id'] == 2) $out['b_state'] = '1->2';
6339: 			
6340: 			}
6341: 			else
6342: 			{
6343: 				$allowed_fields = array(									
6344: 										'c_id'					=>		'id_car',
6345: 										'c_payment_way'			=>		'id_payment_method',
6346: 										'c_payment_card'		=>		'id_payment_card',
6347: 										'c_options'				=>		'options'
6348: 				);
6349: 				$forbidden_fields = array();
6350: 				$affected_fields = array();
6351: 				$affected_keys = array();
6352: 				$filtered_data = array();
6353: 				
6354: 				if (!empty($data))
6355: 				{
6356: 					$data = json_decode($data,true);
6357: 					
6358: 					if (!is_array($data)) 
6359: 					{
6360: 						return $this->showError('404', 'error', 'wrong data');
6361: 					}
6362: 
6363: 					foreach ($data as $key => $value)
6364: 					{
6365: 						if (!empty($allowed_fields[$key]))
6366: 						{
6367: 							if (is_string($value)) $data[$key] = trim($value);
6368: 							$affected_fields[] = $key;		
6369: 							$affected_keys[$key] = true;		
6370: 							$filtered_data[$allowed_fields[$key]] = $data[$key];
6371: 						}
6372: 						else
6373: 						{
6374: 							$forbidden_fields[] = $key;
6375: 						}
6376: 					}
6377: 				}
6378: 
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
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
```

### `setOrderTips` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `1=Client, 2=Driver, 5=Agent`
- source range: `11201-11337`
```php
11201: 		public function setOrderTips($id_order = "", $b_tips = NULL, $c_tips = NULL)
11202: 		{
11203: 			if (empty($_SESSION[UID])) {
11204: 				return $this->showError('404', 'error', 'unauthorized access');
11205: 			}
11206: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
11207: 			{
11208: 				return $this->showError('404', 'error', 'wrong user role');
11209: 			}
11210: 
11211: 			if ($this->id_role == 1 || $this->id_role == 5)
11212: 			{
11213: 				if ($b_tips === NULL) return $this->showError('404', 'error', 'null b_tips');
11214: 				$tips = $b_tips;
11215: 
11216: 				$s = "SELECT
11217: 						`id_order`,
11218: 						`client`,
11219: 						`id_order_status`,
11220: 						`tips`
11221: 					FROM `order` 		
11222: 					WHERE	
11223: 						`id_order` = '" . $id_order . "'
11224: 					LIMIT 1
11225: 					";
11226: 
11227: 				$q = query($s);
11228: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
11229: 
11230: 				$d = fetch_assoc($q);
11231: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
11232: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
11233: 				if ($d['client'] != $_SESSION[UID]) 
11234: 				{
11235: 					return $this->showError('404', 'error', 'user is not author');
11236: 				}
11237: 				if ($d['tips'] !== NULL) return $this->showError('404', 'error', 'c_tips already inputed');
11238: 
11239: 				$s = "UPDATE `order`
11240: 					SET 
11241: 						`tips` = '" . $tips  . "',
11242: 						`last_edit_datetime` = now(),
11243: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
11244: 					WHERE
11245: 						`id_order` = '" . $id_order . "' AND `tips` IS NULL
11246: 					";
11247: 
11248: 				query($s);
11249: 		
11250: 				global $link;
11251: 				if (mysqli_affected_rows($link) === -1) 
11252: 				{
11253: 					return $this->showError('404', 'error', 'database update failed');
11254: 				}
11255: 				elseif (mysqli_affected_rows($link) === 0) 
11256: 				{
11257: 					return $this->showError('404', 'error', 'modified data not found');
11258: 				}
11259: 			}
11260: 			else
11261: 			{
11262: 				if ($c_tips === NULL) return $this->showError('404', 'error', 'null c_tips');
11263: 				$tips = $c_tips;
11264: 
11265: 				$s = "SELECT
11266: 						`order`.`id_order`,
11267: 						`order`.`id_order_status`,
11268: 						od.`id_user`,
11269: 						od.`id_order_driver_status`,
11270: 						od.`tips`
11271: 					FROM `order`
11272: 					LEFT JOIN (
11273: 							SELECT
11274: 								`id_order`,
11275: 								`id_user`,
11276: 								`id_order_driver_status`,
11277: 								`tips`
11278: 							FROM
11279: 								`order_driver`
11280: 							WHERE
11281: 								`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
11282: 						) od USING (`id_order`)				
11283: 					WHERE	
11284: 						`order`.`id_order` = '" . $id_order . "'
11285: 					LIMIT 1
11286: 					";
11287: 
11288: 				$q = query($s);
11289: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
11290: 
11291: 				$d = fetch_assoc($q);
11292: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
11293: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
11294: 
11295: 				if (empty($d['id_user'])) 
11296: 				{
11297: 					return $this->showError('404', 'error', 'user is not performer');
11298: 				}
11299: 
11300: 				if ($d['tips'] !== NULL) return $this->showError('404', 'error', 'b_tips already inputed');
11301: 
11302: 				if ($d['id_order_driver_status'] == 1 || $d['id_order_driver_status'] == 2) 
11303: 				{
11304: 					return $this->showError('404', 'error', 'wrong booking driver state');
11305: 				}
11306: 
11307: 				$s = "UPDATE `order`,`order_driver`
11308: 					SET 
11309: 						`order`.`last_edit_datetime` = now(),
11310: 						`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
11311: 						`order_driver`.`tips` = '" . $tips  . "'
11312: 					WHERE
11313: 						`order`.`id_order` = '" . $id_order . "' AND
11314: 						`order`.`id_order` = `order_driver`.`id_order` AND 
11315: 						`order_driver`.`id_order` = '" . $id_order . "' AND 
11316: 						`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
11317: 						`order_driver`.`tips` IS NULL
11318: 					";
11319: 
11320: 				query($s);
11321: 
11322: 				global $link;
11323: 				if (mysqli_affected_rows($link) === -1) 
11324: 				{
11325: 					return $this->showError('404', 'error', 'driver update failed');
11326: 				}
11327: 				elseif (mysqli_affected_rows($link) === 0) 
11328: 				{
11329: 					return $this->showError('404', 'error', 'driver modified data not found');
11330: 				}
11331: 			}
11332: 
11333: 			return array(
11334: 				'code' 		=>	'200',
11335: 				'status' 	=>	'success'
11336: 			);
11337: 		}
```

### `setTaskStatus` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `23233-23272`
```php
23233: 		public function setTaskStatus($id_task_list = '', $from_status = '', $to_status = '')
23234: 		{
23235: 			if (empty($_SESSION[UID])) {
23236: 				return $this->showError('404', 'error', 'unauthorized access');
23237: 			}
23238: 
23239: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23240: 
23241: 			if (empty($id_task_list)) return $this->showError('404', 'error', 'empty tl_id');
23242: 			$sql_where = "`id_task_list` = '$id_task_list'";
23243: 
23244: 			if (!empty($from_status))
23245: 			{
23246: 				$sql_where .= " AND `id_task_status` = '$from_status'";
23247: 			}
23248: 			if (empty($to_status)) return $this->showError('404', 'error', 'empty to_status');
23249: 			
23250: 			$s = "UPDATE `task_list`
23251: 				SET
23252: 					`id_task_status` = '$to_status'
23253: 				WHERE
23254: 					$sql_where AND `active` = 1
23255: 				";
23256: 
23257: 			$q = query($s);
23258: 			$update = affected_rows();
23259: 			if ($update === -1) 
23260: 			{
23261: 				return $this->showError('400','error','update failed'. error_db());
23262: 			}
23263: 			elseif ($update === 0) 
23264: 			{
23265: 				return $this->showError('400','error','modified data not found');
23266: 			}
23267: 
23268: 			return array(
23269: 				'code' 		=>	'200',
23270: 				'status' 	=>	'success'
23271: 			);
23272: 		}
```

### `startOrder` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver`
- source range: `8248-8337`
```php
8248: 		public function startOrder($id_order = "")
8249: 		{
8250: 			if (empty($_SESSION[UID])) {
8251: 				return $this->showError('404', 'error', 'unauthorized access');
8252: 			}
8253: 			if ($this->id_role != 2)
8254: 			{
8255: 				return $this->showError('404', 'error', 'wrong user role');
8256: 			}
8257: 
8258: 			$s = "SELECT
8259: 					`order`.`id_order`,
8260: 					`order`.`id_order_status`,
8261: 					od.`id_user`,
8262: 					od.`id_order_driver_status`
8263: 				FROM `order` 
8264: 				LEFT JOIN (
8265: 						SELECT
8266: 							`id_order`,
8267: 							`id_user`,
8268: 							`id_order_driver_status`
8269: 						FROM
8270: 							`order_driver`
8271: 						WHERE
8272: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
8273: 					) od USING (`id_order`)				
8274: 				WHERE	
8275: 					`order`.`id_order` = '" . $id_order . "'
8276: 				LIMIT 1
8277: 				";
8278: 
8279: 			$q = query($s);
8280: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8281: 
8282: 			$d = fetch_assoc($q);
8283: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8284: 			if ($d['id_order_status'] != 2) 
8285: 			{
8286: 				return $this->showError('404', 'error', 'wrong booking state');
8287: 			}
8288: 			if (empty($d['id_user'])) 
8289: 			{
8290: 				return $this->showError('404', 'error', 'user is not performer');
8291: 			}
8292: 			
8293: 			if ($d['id_order_driver_status'] == 1) 
8294: 			{
8295: 				return $this->showError('404', 'error', 'not appointed performer');
8296: 			}
8297: 			elseif ($d['id_order_driver_status'] == 2)
8298: 			{
8299: 				return $this->showError('404', 'error', 'canceled performer');
8300: 			}
8301: 			elseif ($d['id_order_driver_status'] == 5)
8302: 			{
8303: 				return $this->showError('404', 'error', 'already started ride');
8304: 			}
8305: 			elseif ($d['id_order_driver_status'] == 6)
8306: 			{
8307: 				return $this->showError('404', 'error', 'completed booking');
8308: 			}
8309: 			
8310: 			$s = "UPDATE `order`,`order_driver`
8311: 				SET 
8312: 					`order`.`last_edit_datetime` = now(),
8313: 					`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
8314: 					`order_driver`.`id_order_driver_status` = '5',
8315: 					`order_driver`.`start_datetime` = now()
8316: 				WHERE
8317: 					`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order` = `order_driver`.`id_order` AND
8318: 					`order_driver`.`id_order` = '" . $id_order . "' AND 
8319: 					`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
8320: 					`order_driver`.`id_order_driver_status` in (3,4)
8321: 				";
8322: 
8323: 			query($s);
8324: 			global $link;
8325: 			if (mysqli_affected_rows($link) === -1) 
8326: 			{
8327: 				return $this->showError('404', 'error', 'database update failed');
8328: 			}
8329: 			elseif (mysqli_affected_rows($link) === 0) 
8330: 			{
8331: 				return $this->showError('404', 'error', 'modified data not found');
8332: 			}
8333: 			return array(
8334: 				'code' 		=>	'200',
8335: 				'status' 	=>	'success'
8336: 			);
8337: 		}
```

### `statusCartBlock` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `2=Driver, 4=Administrator`
- source range: `20341-20501`
```php
20341: 		public function statusCartBlock($data = "")
20342: 		{	
20343: 			if (empty($_SESSION[UID])) {
20344: 				return $this->showError('404', 'error', 'unauthorized access');
20345: 			}
20346: 
20347: 			if ($this->id_role != 4)
20348: 			{
20349: 				if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong user role');
20350: 				if ($_SESSION['id_verification_status'] != 2)
20351: 				{
20352: 					return $this->showError('404', 'error', 'wrong user check state');
20353: 				}
20354: 			}
20355: 
20356: 			if (empty($data)) 
20357: 			{
20358: 				return $this->showError('404', 'error', 'empty data');
20359: 			}
20360: 
20361: 			$data = json_decode($data,true);
20362: 			
20363: 			if (empty($data) || !is_array($data)) 
20364: 			{
20365: 				return $this->showError('404', 'error', 'wrong data');
20366: 			}
20367: 
20368: 			$out = array(
20369: 				'code' 		=>	'200',
20370: 				'status' 	=>	'success'
20371: 			);		
20372: 			foreach($data as $i=>$c_b_arr)
20373: 			{
20374: 				if (!isset($c_b_arr['u_id'])) continue;
20375: 				$id_user = $c_b_arr['u_id'];
20376: 				if (!isset($c_b_arr['sc_id'])) continue;
20377: 				$product = $c_b_arr['sc_id'];
20378: 				if (!isset($c_b_arr['block'])) continue;
20379: 				$property = $c_b_arr['block'];
20380: 				$filtered_data = array();
20381: 				if (array_key_exists('status',$c_b_arr))
20382: 				{
20383: 					$filtered_data['status'] = $c_b_arr['status'];
20384: 				}
20385: 				if (isset($c_b_arr['status_sys']))
20386: 				{
20387: 					$filtered_data['status_sys'] = $c_b_arr['status_sys'];
20388: 				}
20389: 				$s = "SELECT 
20390: 						`id_cart_block`
20391: 					FROM `cart_block`
20392: 					WHERE
20393: 						`id_user` = '$id_user' AND `product` = '$product' AND `property` = '$property'
20394: 					LIMIT 1
20395: 					";		
20396: 
20397: 				$q = query($s);
20398: 				if ($q === false)
20399: 				{
20400: 					$out['warning'][]  = "$i cart_block select failed";
20401: 				}
20402: 				else
20403: 				{
20404: 					$d = fetch_assoc($q);
20405: 					if ($d === NULL)
20406: 					{
20407: 						$out['warning'][]  = "$i cart_block not found";
20408: 					}
20409: 					$id_cart_block = $d['id_cart_block'];
20410: 				}
20411: 				if (!empty($filtered_data)) 
20412: 				{
20413: 					if ($this->id_role != 4) {$out['warning'][]  = "$i update wrong user role"; continue;}
20414: 	
20415: 					$s = array();
20416: 					foreach ($filtered_data as $key => $value)
20417: 					{
20418: 					
20419: 						$s[] = "`" . $key . "` = " 
20420: 								   . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
20421: 					}
20422: 
20423: 					$s = "UPDATE `cart_block`
20424: 						SET 
20425: 							" . implode(",\n", $s) ."
20426: 						WHERE
20427: 							`id_cart_block` = '$id_cart_block'
20428: 						";
20429: 
20430: 					$q = query($s);
20431: 
20432: 					if ($q === false) $out['warning'][]  = "$i update failed";	
20433: 				}
20434: 				if (!empty($c_b_arr['statuses']) && is_array($c_b_arr['statuses']))
20435: 				{
20436: 					foreach($c_b_arr['statuses'] as $j=>$c_b_arr_statuses)
20437: 					{
20438: 						if (!isset($c_b_arr_statuses['t_id'])) continue;
20439: 						$id_trip = $c_b_arr_statuses['t_id'];
20440: 
20441: 						$s = "SELECT 
20442: 								`driver`
20443: 							FROM `trip`
20444: 							WHERE
20445: 								`id_trip` = '$id_trip'
20446: 							LIMIT 1
20447: 							";		
20448: 
20449: 						$q = query($s);
20450: 						if ($q === false)
20451: 						{
20452: 							$out['warning'][]  = "$i $j trip select failed";
20453: 						}
20454: 						else
20455: 						{
20456: 							$d = fetch_assoc($q);
20457: 							if ($d === NULL)
20458: 							{
20459: 								$out['warning'][]  = "$i $j trip not found";
20460: 								continue;
20461: 							}
20462: 							$driver = $d['driver'];
20463: 							if ($driver != $_SESSION[UID] && $this->id_role != 4)
20464: 							{
20465: 								$out['warning'][]  = "$i $j foreign trip";
20466: 								continue;
20467: 							}
20468: 						}
20469: 						$filtered_data = array();
20470: 						if (array_key_exists('status',$c_b_arr_statuses))
20471: 						{
20472: 							$filtered_data['status'] = $c_b_arr_statuses['status'];
20473: 						}
20474: 						if (isset($c_b_arr_statuses['status_sys']))
20475: 						{
20476: 							$filtered_data['status_sys'] = $c_b_arr_statuses['status_sys'];
20477: 						}
20478: 						$s = array();
20479: 						foreach ($filtered_data as $key => $value)
20480: 						{
20481: 						
20482: 							$s[] = "`" . $key . "` = " 
20483: 									   . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
20484: 						}
20485: 
20486: 						$s = "UPDATE `cart_block_trip`
20487: 							SET 
20488: 								" . implode(",\n", $s) ."
20489: 							WHERE
20490: 								`id_cart_block` = '$id_cart_block' AND `id_trip` = '$id_trip'
20491: 							";
20492: 
20493: 						$q = query($s);
20494: 
20495: 						if ($q === false) $out['warning'][]  = "$i $j  update failed";
20496: 					}
20497: 				}
20498: 			}
20499: 
20500: 			return $out;
20501: 		}
```

### `transferCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `25725-26023`
```php
25725: 		public function transferCurrencyAccount($data = '', $account_currency_list = array())
25726: 		{		
25727: 			if (empty($_SESSION[UID])) {
25728: 				return $this->showError('404', 'error', 'unauthorized access');
25729: 			}
25730: 
25731: 			if (empty($data)) 
25732: 			{
25733: 				return $this->showError('404', 'error', 'empty data');
25734: 			}
25735: 
25736: 			$data = json_decode($data,true);
25737: 			
25738: 			if (empty($data) || !is_array($data)) 
25739: 			{
25740: 				return $this->showError('404', 'error', 'wrong data');
25741: 			}
25742: 
25743: 			$allowed_fields = array(
25744: 				'sender'	=>		array(
25745: 										'name'	=>	false,
25746: 										'desc'	=>	''
25747: 									),
25748: 				'recipient'	=>		array(
25749: 										'name'	=>	false,
25750: 										'desc'	=>	''
25751: 									),
25752: 				'from'	=>		array(
25753: 										'name'	=>	'from',
25754: 										'NULL'	=>	true
25755: 									),
25756: 				'to'	=>		array(
25757: 										'name'	=>	'to',
25758: 										'NULL'	=>	true
25759: 									),
25760: 				'sum'	=>		array(
25761: 										'name'	=>	'sum',
25762: 										'NULL'	=>	true
25763: 									),
25764: 				'currency'	=>		array(
25765: 										'name'	=>	'currency',
25766: 										'NULL'	=>	true
25767: 									),
25768: 				'json'	=>		array(
25769: 										'name'	=>	'json',
25770: 										'NULL'	=>	false
25771: 									)
25772: 			);
25773: 
25774: 			$forbidden_fields = array();
25775: 			$affected_fields = array();
25776: 			$filtered_data = array();
25777: 			$add_table_list = array();
25778: 			foreach ($data as $key => $value)
25779: 			{
25780: 				if (isset($allowed_fields[$key]))
25781: 				{
25782: 					if (!empty($allowed_fields[$key]['format']))
25783: 					{
25784: 						$value = $allowed_fields[$key]['format']($value);
25785: 					}
25786: 					if (empty($value['error']))
25787: 					{
25788: 						if ($allowed_fields[$key]['name'] === NULL)
25789: 						{
25790: 							$add_table_list[] = array(
25791: 								$allowed_fields[$key]['table'],
25792: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
25793: 								$value,
25794: 								$key,
25795: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
25796: 								empty($data['exact']) ? false : true
25797: 							);
25798: 						}
25799: 						elseif (isset($allowed_fields[$key]['desc']))
25800: 						{
25801: 							continue;
25802: 						}
25803: 						else
25804: 						{									
25805: 							$name = $allowed_fields[$key]['name'];
25806: 							$null_on = $allowed_fields[$key]['NULL'];
25807: 							if ($null_on === true || $value !== NULL)
25808: 							{
25809: 								$affected_fields[] = $key;
25810: 								$filtered_data[$name] = $value;
25811: 							}
25812: 							else
25813: 							{
25814: 								return $this->showError('404', 'error', "$key: null value");
25815: 							}
25816: 						}
25817: 					}
25818: 					else
25819: 					{
25820: 						return $this->showError('404', 'error', "$key: {$value['error']}");
25821: 					}
25822: 				}
25823: 				else
25824: 				{
25825: 					$forbidden_fields[] = $key;
25826: 				}
25827: 			}
25828: 
25829: 			if (empty($filtered_data['sum'])) return $this->showError('404', 'error', 'empty sum');
25830: 			if (empty($filtered_data['currency'])) $filtered_data['currency'] = DEFAULT_CURRENCY;
25831: 			if (!isset($account_currency_list[$filtered_data['currency']])) return $this->showError('404', 'error', 'wrong currency');
25832: 			if (empty($filtered_data['to']) && empty($data['recipient'])) return $this->showError('404', 'error', 'empty recipient data');
25833: 
25834: 			$q = query("BEGIN");
25835: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
25836: 
25837: 			if (empty($filtered_data['from']))
25838: 			{
25839: 				if (empty($data['sender'])) 
25840: 				{
25841: 					$data['sender'] = $_SESSION[UID];
25842: 				}
25843: 				else
25844: 				{
25845: 					if ($this->id_role != 4 && $data['sender'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25846: 				}
25847: 
25848: 				$s = "SELECT			
25849: 					  `id_currency_account`,
25850: 					  `id_user`,
25851: 					  `id_currency_account_status`,
25852: 					  `sum`
25853: 					FROM `currency_account`
25854: 					WHERE	
25855: 						`id_user` = '{$data['sender']}' AND `currency` = '{$filtered_data['currency']}'
25856: 					LIMIT 1
25857: 					FOR UPDATE
25858: 					";
25859: 
25860: 				$q = query($s);
25861: 				if ($q === false) return $this->showError('404', 'error', 'sender select failed');
25862: 				$d = fetch_assoc($q);
25863: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'sender account not found');
25864: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong sender account currency');
25865: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong sender account status');
25866: 				if ((int)$d['sum'] < (int)$filtered_data['sum']) return $this->showError('404', 'error', 'too much sum for sender');
25867: 				$filtered_data['from'] = $d['id_currency_account'];			
25868: 			}
25869: 			else
25870: 			{
25871: 				$s = "SELECT			
25872: 					  `id_currency_account`,
25873: 					  `id_user`,
25874: 					  `currency`,
25875: 					  `id_currency_account_status`,
25876: 					  `sum`
25877: 					FROM `currency_account`
25878: 					WHERE	
25879: 						`id_currency_account` = '{$filtered_data['from']}'
25880: 					LIMIT 1
25881: 					FOR UPDATE
25882: 					";
25883: 
25884: 				$q = query($s);
25885: 				if ($q === false) return $this->showError('404', 'error', 'from select failed');
25886: 				$d = fetch_assoc($q);
25887: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'from account not found');
25888: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'from: not enough rights');
25889: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong from account currency');
25890: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong from account status');
25891: 				if ((int)$d['sum'] < (int)$filtered_data['sum']) return $this->showError('404', 'error', 'too much sum');
25892: 			}
25893: 
25894: 			$out = array('affected_fields' 	=> 	$affected_fields);
25895: 			if (empty($filtered_data['to']))
25896: 			{
25897: 				if ($data['recipient'] == $_SESSION[UID]) return $this->showError('404', 'error', 'recipient is sender');
25898: 				$s = "SELECT			
25899: 					  `id_currency_account`,
25900: 					  `id_user`,
25901: 					  `id_currency_account_status`
25902: 					FROM `currency_account`
25903: 					WHERE	
25904: 						`id_user` = '{$data['recipient']}' AND `currency` = '{$filtered_data['currency']}'
25905: 					LIMIT 1
25906: 					FOR UPDATE
25907: 					";
25908: 
25909: 				$q = query($s);
25910: 				if ($q === false) return $this->showError('404', 'error', 'recipient select failed');
25911: 				$d = fetch_assoc($q);
25912: 				if (empty($d['id_currency_account']))
25913: 				{
25914: 					$now_time = time();
25915: 					$s = "INSERT INTO `currency_account`
25916: 						SET 
25917: 						  `id_user` = '{$data['recipient']}',
25918: 						  `sum`= 0,
25919: 						  `currency` = '{$filtered_data['currency']}',
25920: 						  `reserved` = 0,
25921: 						  `create_user` = '{$_SESSION[UID]}',
25922: 						  `last_edit_user` = '{$_SESSION[UID]}',
25923: 						  `last_edit_int_timestamp` = '$now_time',
25924: 						  `create_int_timestamp` = '$now_time'
25925: 						";
25926: 
25927: 					$q = query($s);
25928: 					if ($q === false) return $this->showError('404', 'error', 'insert failed');
25929: 					$a_id = insert_id();
25930: 					$out['a_id'] = $a_id;
25931: 					$filtered_data['to'] = $a_id;
25932: 				}
25933: 				else
25934: 				{					
25935: 					if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong to account status');
25936: 					$filtered_data['to'] = $d['id_currency_account'];	
25937: 				}
25938: 				if ($filtered_data['from'] == $filtered_data['to']) return $this->showError('404', 'error', 'same from, to');
25939: 			}
25940: 			else
25941: 			{
25942: 				if ($filtered_data['from'] == $filtered_data['to']) return $this->showError('404', 'error', 'same from and to');
25943: 				$s = "SELECT			
25944: 					  `id_currency_account`,
25945: 					  `id_user`,
```

### `translate` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `20301-20339`
```php
20301: 		public function translate($data = "", $from = "ru", $to = "en")
20302: 		{	
20303: 			if (empty($_SESSION[UID])) {
20304: 				if (empty($_SESSION['edit_langs'])) return $this->showError('404', 'error', 'unauthorized access');
20305: 			} else {
20306: 				if ($this->id_role != 4)
20307: 				{
20308: 					return $this->showError('404', 'error', 'not enough rights');
20309: 				}
20310: 			}
20311: 
20312: 			if (empty($from)) $from = "ru";
20313: 			if (empty($to)) $to = "ru";
20314: 			
20315: 			$out = array();
20316: 			if (!empty($data))
20317: 			{
20318: 				$data = json_decode($data,true);
20319: 
20320: 				if (empty($data) || !is_array($data)) 
20321: 				{
20322: 					return $this->showError('404', 'error', 'wrong data');
20323: 				}
20324: 				$formated = array();
20325: 				foreach($data as $str)
20326: 				{		
20327: 					@$formated[] = (string)$str;
20328: 				}
20329: 				$out = batch_translate($formated,$from,$to);
20330: 				if (!empty($out['error'])) return $this->showError('404', 'error', $out['error']);
20331: 			}
20332: 
20333: 			return array(
20334: 				'code' 		=>	'200',
20335: 				'status' 	=>	'success',
20336: 				'data'		=>	$out,
20337: 				'translator'=>	2
20338: 			);
20339: 		}
```

### `withdrawCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `25450-25723`
```php
25450: 		public function withdrawCurrencyAccount($data = '', $payment_services = array(), $payment_ways = array(), $account_currency_list = array())
25451: 		{
25452: 			if (empty($_SESSION[UID])) {
25453: 				return $this->showError('404', 'error', 'unauthorized access');
25454: 			}
25455: 
25456: 			if (empty($data)) 
25457: 			{
25458: 				return $this->showError('404', 'error', 'empty data');
25459: 			}
25460: 
25461: 			$data = json_decode($data,true);
25462: 			
25463: 			if (empty($data) || !is_array($data)) 
25464: 			{
25465: 				return $this->showError('404', 'error', 'wrong data');
25466: 			}
25467: 
25468: 			$allowed_fields = array(
25469: 				'card_number'	=>		array(
25470: 										'name'	=>	false,
25471: 										'desc'	=>	''
25472: 									),
25473: 				
25474: 				'a_id'	=>		array(
25475: 										'name'	=>	false,
25476: 										'desc'	=>	'id_currency_account'
25477: 									),			
25478: 				'sum'	=>		array(
25479: 										'name'	=>	'sum',
25480: 										'NULL'	=>	true
25481: 									),
25482: 				'currency'	=>		array(
25483: 										'name'	=>	'currency',
25484: 										'NULL'	=>	true
25485: 									),
25486: 									
25487: 				'payment_service'	=>		array(
25488: 										'name'	=>	false,
25489: 										'desc'	=>	'id_payment_service'
25490: 									),
25491: 				'payment_way'	=>		array(
25492: 										'name'	=>	false,
25493: 										'desc'	=>	'id_payment_method'
25494: 									),
25495: 				'json'	=>		array(
25496: 										'name'	=>	'json',
25497: 										'NULL'	=>	false
25498: 									)
25499: 			);
25500: 
25501: 			$forbidden_fields = array();
25502: 			$affected_fields = array();
25503: 			$filtered_data = array();
25504: 			$add_table_list = array();
25505: 			foreach ($data as $key => $value)
25506: 			{
25507: 				if (isset($allowed_fields[$key]))
25508: 				{
25509: 					if (!empty($allowed_fields[$key]['format']))
25510: 					{
25511: 						$value = $allowed_fields[$key]['format']($value);
25512: 					}
25513: 					if (empty($value['error']))
25514: 					{
25515: 						if ($allowed_fields[$key]['name'] === NULL)
25516: 						{
25517: 							$add_table_list[] = array(
25518: 								$allowed_fields[$key]['table'],
25519: 								empty($allowed_fields[$key]['key']) ? array() : $allowed_fields[$key]['key'],
25520: 								$value,
25521: 								$key,
25522: 								empty($allowed_fields[$key]['allowed_fields']) ? array() : $allowed_fields[$key]['allowed_fields'],
25523: 								empty($data['exact']) ? false : true
25524: 							);
25525: 						}
25526: 						elseif (isset($allowed_fields[$key]['desc']))
25527: 						{
25528: 							continue;
25529: 						}
25530: 						else
25531: 						{									
25532: 							$name = $allowed_fields[$key]['name'];
25533: 							$null_on = $allowed_fields[$key]['NULL'];
25534: 							if ($null_on === true || $value !== NULL)
25535: 							{
25536: 								$affected_fields[] = $key;
25537: 								$filtered_data[$name] = $value;
25538: 							}
25539: 							else
25540: 							{
25541: 								return $this->showError('404', 'error', "$key: null value");
25542: 							}
25543: 						}
25544: 					}
25545: 					else
25546: 					{
25547: 						return $this->showError('404', 'error', "$key: {$value['error']}");
25548: 					}
25549: 				}
25550: 				else
25551: 				{
25552: 					$forbidden_fields[] = $key;
25553: 				}
25554: 			}
25555: 
25556: 			if (empty($data['card_number'])) return $this->showError('404', 'error', 'empty card number');
25557: 			if (empty($filtered_data['sum'])) return $this->showError('404', 'error', 'empty sum');
25558: 			if (empty($filtered_data['currency'])) $filtered_data['currency'] = DEFAULT_CURRENCY;
25559: 			if (!isset($account_currency_list[$filtered_data['currency']])) return $this->showError('404', 'error', 'wrong currency');
25560: 
25561: 			if (empty($data['payment_service'])) $data['payment_service'] = 1;
25562: 			if (empty($payment_services[$data['payment_service']])) return $this->showError('404', 'error', 'wrong payment service');
25563: 			if (empty($payment_services[$data['payment_service']]['var'])) return $this->showError('404', 'error', 'empty payment service var');
25564: 			
25565: 			$f_name = "{$payment_services[$data['payment_service']]['var']}_create_payout";		
25566: 			if (!function_exists($f_name)) return $this->showError('404', 'error', 'function not found');
25567: 			if (empty($data['payment_way'])) $data['payment_way'] = 2;
25568: 			if (empty($payment_ways[$data['payment_way']])) return $this->showError('404', 'error', 'wrong payment way');			
25569: 			foreach($payment_services[$data['payment_service']]['payment_ways'] as $key=>$val)
25570: 			{
25571: 				if ($val[0] == $data['payment_way']) 
25572: 				{
25573: 					$p_w_index = $key;	
25574: 					if (!empty($val[1]['api']['type'])) $p_w_type =  $val[1]['api']['type'];
25575: 				}
25576: 			}
25577: 			if (!isset($p_w_index)) return $this->showError('404', 'error', 'forbidden payment way');
25578: 			if (!isset($p_w_type)) return $this->showError('404', 'error', 'empty payment way type');
25579: 
25580: 			$q = query("BEGIN");
25581: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
25582: 
25583: 			if (empty($data['a_id'])) 
25584: 			{
25585: 				$s = "SELECT			
25586: 					  `id_currency_account`,
25587: 					  `id_user`,
25588: 					  `currency`,
25589: 					  `id_currency_account_status`,
25590: 					  `sum`
25591: 					FROM `currency_account`
25592: 					WHERE	
25593: 						`id_user` = '{$_SESSION[UID]}' AND `currency` = '{$filtered_data['currency']}'
25594: 					LIMIT 1
25595: 					FOR UPDATE
25596: 					";
25597: 
25598: 				$q = query($s);
25599: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
25600: 				$d = fetch_assoc($q);
25601: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'account not found');
25602: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
25603: 				if ((int)$d['sum'] < (int)$filtered_data['sum']) return $this->showError('404', 'error', 'too much sum');
25604: 				$a_id = $d['id_currency_account'];
25605: 			}
25606: 			else
25607: 			{
25608: 				$s = "SELECT			
25609: 					  `id_currency_account`,
25610: 					  `id_user`,
25611: 					  `currency`,
25612: 					  `id_currency_account_status`,
25613: 					  `sum`
25614: 					FROM `currency_account`
25615: 					WHERE	
25616: 						`id_currency_account` = '{$data['a_id']}'
25617: 					LIMIT 1
25618: 					FOR UPDATE
25619: 					";
25620: 
25621: 				$q = query($s);
25622: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
25623: 				$d = fetch_assoc($q);
25624: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'currency account not found');
25625: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25626: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
25627: 				if ($d['id_currency_account_status'] != 1) return $this->showError('404', 'error', 'wrong account status');
25628: 				if ((int)$d['sum'] < (int)$filtered_data['sum']) return $this->showError('404', 'error', 'too much sum');
25629: 				$a_id = $data['a_id'];
25630: 			}
25631: 
25632: 			$s = array();
25633: 			foreach ($filtered_data as $key => $value)
25634: 			{
25635: 				$s[] = "`$key` = " 
25636: 					 . ($value === NULL ? "NULL" : "'" . real_escape_string(is_array($value) ? json_encode($value) : $value) . "'");
25637: 			}
25638: 
25639: 			$total_sum = ((float)$filtered_data['sum']) * (1  - ($this->constant['commission_withdraw']) / 100);
25640: 			$now_time = time();
25641: 			$s = "INSERT INTO `transaction`
25642: 				SET 
25643: 				  " . implode(",\n", $s) .",
25644: 				  `total_sum` = '$total_sum',
25645: 				  `from` = '$a_id',
25646: 				  `commission` = '{$this->constant['commission_withdraw']}',
25647: 				  `id_transaction_type` = '2',
25648: 				  `create_user` = '{$_SESSION[UID]}',
25649: 				  `last_edit_user` = '{$_SESSION[UID]}',
25650: 				  `last_edit_int_timestamp` = '$now_time',
25651: 				  `create_int_timestamp` = '$now_time'
25652: 				";
25653: 
25654: 			$q = query($s);
25655: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');	
25656: 			
25657: 			$trn_id = insert_id();
25658: 			$create_status = $f_name($total_sum,$filtered_data['currency'],$data['card_number'],array('trn_id'=>$trn_id));
25659: 			if (!empty($create_status['error'])) return $this->showError('404', 'error', $create_status['error']);
25660: 
25661: 			$arr = array(
25662: 					'pending'				=>		'1',
25663: 					'succeeded'				=>		'2',
25664: 					'canceled'				=>		'3'		
25665: 			);
25666: 
25667: 			$s = "UPDATE `currency_account`
25668: 				SET
25669: 					`sum` = `sum` - '{$filtered_data['sum']}',
25670: 					`last_edit_user` = '{$_SESSION[UID]}',
```

### `writeTicket` — `archive_17012026_1259/taxi/models/api.php`
- role IDs: `4=Administrator`
- source range: `18049-18231`
```php
18049: 		public function writeTicket($id_trip = "", $data = "")
18050: 		{
18051: 			if (empty($_SESSION[UID])) {
18052: 				return $this->showError('404', 'error', 'unauthorized access');
18053: 			}
18054: 
18055: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
18056: 
18057: 			if (empty($data)) 
18058: 			{
18059: 				return $this->showError('404', 'error', 'empty data');
18060: 			}
18061: 
18062: 			$data = json_decode($data,true);
18063: 			
18064: 			if (empty($data) || !is_array($data)) 
18065: 			{
18066: 				return $this->showError('404', 'error', 'wrong data');
18067: 			}
18068: 			
18069: 			$filtered_data = array();
18070: 			$seats = array();
18071: 			$folder = $_SERVER['DOCUMENT_ROOT'] . CONFIG_USER_FILE_PATH . "trips/$id_trip/ticket/";
18072: 			foreach($data as $i=>$file_arr)
18073: 			{
18074: 				if(!is_array($file_arr)) return $this->showError('404', 'error', "$i: wrong data");
18075: 				if (isset($file_arr['base64']))
18076: 				{
18077: 					if (empty($file_arr['seat'])) return $this->showError('404', 'error', "$i: empty seat");
18078: 					$seat = trim($file_arr['seat']);
18079: 					$seats[] = $seat;
18080: 					$filtered_data[$i]['seat'] = $seat;
18081: 					$val = trim($file_arr['base64']);
18082: 					if ($val === '')
18083: 					{
18084: 						$filtered_data[$i]['blob_link'] = '';
18085: 						$filtered_data[$i]['blob_mime'] = '';
18086: 						$filtered_data[$i]['blob'] = false;
18087: 						$filtered_data[$i]['blob_ext_w._dot'] = '';
18088: 						$filtered_data[$i]['path'] = "$folder";	
18089: 					}
18090: 					else
18091: 					{
18092: 						$filtered_data[$i]['blob_link'] = $seat;
18093: 						@$content = (string)$val;
18094: 						list($type, $content) = array_merge(explode(';', $content),array(''));
18095: 						list(, $type) = array_merge(explode(':', $type),array(''));
18096: 						if (empty($type)) return $this->showError('404', 'error', 'wrong file string');
18097: 						list($base64, $content) = array_merge(explode(',', $content),array(''));
18098: 						if ($base64 !== 'base64') return $this->showError('404', 'error', 'wrong string of file');
18099: 						$content = base64_decode($content);
18100: 						if (gettype($content) !== 'string') return $this->showError('404', 'error', 'wrong base64');
18101: 						$filtered_data[$i]['blob_mime'] = $type;
18102: 						$filtered_data[$i]['blob'] = $content;
18103: 						$filtered_data[$i]['blob_ext_w._dot'] = isset($file_arr['extW.Dot']) ? trim($file_arr['extW.Dot']) : '';
18104: 						$filtered_data[$i]['path'] = "$folder";	
18105: 					}
18106: 				}
18107: 				else
18108: 				{
18109: 					return $this->showError('404', 'error', "$i: base64 not found");
18110: 				}
18111: 			}
18112: 
18113: 			$s = "SELECT			
18114: 					`trip`.`id_trip`,
18115: 					`trip`.`driver`,
18116: 					`trip`.`json`,
18117: 					COUNT(`ticket`.`id_seat`) as seats_count,				
18118: 					GROUP_CONCAT(
18119: 						CONCAT_WS(0x00,
18120: 							`ticket`.`id_seat`,
18121: 							`ticket`.`id_trip_seat`
18122: 						)
18123: 					SEPARATOR 0x01) as fix_seats			
18124: 				FROM `trip`
18125: 				LEFT JOIN `ticket` ON `ticket`.`id_trip` = `trip`.`id_trip` AND `ticket`.`id_seat` in ('" . implode("','",$seats) . "')
18126: 				WHERE	
18127: 					`trip`.`id_trip` = '" . $id_trip . "'
18128: 				LIMIT 1
18129: 				";
18130: 			
18131: 			$q = query($s);
18132: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
18133: 
18134: 			$d = fetch_assoc($q);
18135: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18136: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18137: 			{
18138: 				return $this->showError('404', 'error', 'user is not author');
18139: 			}
18140: 
18141: 			$t_options = json_decode($d['json'],true);
18142: 			
18143: 			if (empty($t_options['seats_sold']))
18144: 			{
18145: 				if ((int)$d['seats_count'] != count($seats)) return $this->showError('404', 'error', "seat not found");
18146: 			}
18147: 			else
18148: 			{
18149: 				foreach($seats as $seat)
18150: 				{
18151: 					if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found");	
18152: 				}
18153: 			}
18154: 			$fix_seats = array();
18155: 			if (!empty($d['fix_seats']))
18156: 			{
18157: 				$fix_seats = explode(chr(1),$d['fix_seats']);
18158: 				$d['fix_seats'] = array();
18159: 				foreach ($fix_seats as $key=>$value)
18160: 				{
18161: 					list(
18162: 						$id_seat,
18163: 						$id_trip_seat
18164: 						)= explode(chr(0),$value);
18165: 					$d['fix_seats'][$id_seat] = $id_trip_seat;
18166: 				}
18167: 				$fix_seats = $d['fix_seats'];
18168: 			}
18169: 
18170: 			if (is_file($folder))
18171: 			{
18172: 				return $this->showError('404', 'error', "trips/$id_trip/ticket/ is file");
18173: 			}
18174: 			else
18175: 			{
18176: 				if (!file_exists($folder)) 
18177: 				{
18178: 					@$dir = mkdir($folder,0777,true);
18179: 					if (!$dir) return $this->showError('404', 'error', "trips/$id_trip/ticket/ create error");
18180: 				}
18181: 			}
18182: 
18183: 			$htaccess = $_SERVER['DOCUMENT_ROOT'] . CONFIG_USER_FILE_PATH . '.htaccess';
18184: 			if (!file_exists($htaccess))
18185: 			{
18186: 				@$file = file_put_contents($htaccess,'deny from all');
18187: 				if (!$file) return $this->showError('404', 'error', "htaccess write error");
18188: 			}
18189: 
18190: 			$out = array();
18191: 
18192: 			foreach($filtered_data as $i=>$arr)
18193: 			{
18194: 				$arr['path'] = $arr['path'] . $fix_seats[$arr['seat']];
18195: 				if ($arr['blob'] === false)
18196: 				{
18197: 					if (!@unlink($arr['path']))
18198: 					{
18199: 						$out['warning'][] = "{$arr['seat']} delete error";
18200: 						continue;
18201: 					}
18202: 				}
18203: 				else
18204: 				{
18205: 					@$file = file_put_contents($arr['path'],$arr['blob']);
18206: 					if ($file === false) 
18207: 					{
18208: 						$out['warning'][] = "{$arr['seat']} write error";
18209: 						continue;
18210: 					}
18211: 				}
18212: 
18213: 				$s = "UPDATE `ticket`
18214: 					SET
18215: 						`blob_link` = '" . $fix_seats[$arr['seat']] . "',
18216: 						`blob_mime` = '" . $arr['blob_mime'] . "',
18217: 						`blob_ext_w._dot` = '" . $arr['blob_ext_w._dot'] . "'
18218: 					WHERE
18219: 						`id_trip` = '" . $id_trip . "' AND `id_seat` = '" . $arr['seat'] . "'
18220: 					";
18221: 
18222: 				$q = query($s);
18223: 				if ($q === false) $out['warning'][] = "{$arr['seat']} update failed";
18224: 			}
18225: 
18226: 			return array(
18227: 				'code' 		=>	'200',
18228: 				'status' 	=>	'success',
18229: 				'data'		=>	$out
18230: 			);
18231: 		}
```

## 5. Semantic interpretation
В этой версии не создаётся автоматическая ALLOW/REJECT строка для каждого comparison. `CONFIRMED` decision присваивается только при явном access-denied / forbidden / wrong-role return или эквивалентном control-flow.

### `/query`
Предыдущие RP уже доказали:
```text
Administrator → CAN_EXECUTE → /query
```
при текущем `query_roles = 4`.

### Остальные operations
RP-18 даёт function-level evidence, но каждую branch необходимо нормализовать отдельно до semantic decision.

## 6. Текущая Semantic Matrix
| Role | Operation | Decision | Preconditions | Status |
|---|---|---|---|---|
| Administrator | `/query` | ALLOW | `query_roles` contains 4 | CONFIRMED |
| Client | `/query` | REJECT | current `query_roles=4` | CONFIRMED, snapshot-scoped |
| Driver | `/query` | REJECT | current `query_roles=4` | CONFIRMED, snapshot-scoped |
| Agent | `/query` | REJECT | current `query_roles=4` | CONFIRMED, snapshot-scoped |
| Usher | `/query` | REJECT | current `query_roles=4` | CONFIRMED, snapshot-scoped |
| Usher with extended powers | `/query` | REJECT | current `query_roles=4` | CONFIRMED, snapshot-scoped |

## 7. Методологический результат
Исследование перешло от syntactic evidence:
```text
grep → id_role
```
к operational semantics:
```text
function → branch → decision
```

## 8. Gap Report
```text
G-18-01  normalize non-query role-sensitive functions     OPEN
G-18-02  complete Role × Operation matrix                 OPEN
G-18-03  identify shared authorization helpers            OPEN
G-18-04  map frontend-consumed authorization capabilities  OPEN
```

## 9. MCR
`MCR = NO CHANGE`.

## 10. Следующий шаг
Взять 2–3 function contexts с наиболее явным отказом и довести каждый до semantic decision. После выявления повторяющегося authorization pattern проверить общий helper, а не повторять полный grep.