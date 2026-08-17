# Backend Semantic Graph — Research Pass 20
# Authorization Helper Trace v0.1

**Статус:** CONFIRMED / PARTIALLY RESOLVED
**Методология:** Semantic Graph Research Methodology v2.3
**Предшествующий проход:** RP-19 Reject Branch Normalization v0.1
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Research Question

> Являются ли разобранные role-sensitive checks реализациями одного общего authorization helper или независимыми локальными gates?

## 2. Результат поиска

Найдено helper-related contexts: **62**.
Контекстов, содержащих явное определение известных authorization helper names: **1**.

### Найденные helper-related contexts
#### `archive_17012026_1259/taxi/index.php:14`
```php
9: 		if (!isset($_COOKIE['vfoc']) || $_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))))){session_unset();}
10: 	}
11: 	if (isset($_POST['u_a_id']) || isset($_POST['u_a_email']) || isset($_POST['u_a_phone']) || isset($_POST['u_a_tg']) || isset($_POST['u_a_wa']))
12: 	{
13: 		require_once('models/m_functions.php');
14: 		check_auth_user();
15: 		if (empty($_SESSION[UID]))
16: 		{
17: 			show_error('unauthorized access');
18: 		}
19: 		elseif ($_SESSION['id_role'] != 4)
20: 		{
21: 			show_error('not enough rights');
22: 		}
```

#### `archive_17012026_1259/taxi/models/m_functions.php:247`
```php
242: 			  'order'		=> isset($d['o']) ? $d['o'] :  NULL,
243: 			  'blog_topic'	=> isset($d['bt']) ? $d['bt'] :  NULL,
244: 			  'blog_post'	=> isset($d['bp']) ? $d['bp'] :  NULL
245: 		);
246: 	}
247: 	function check_auth_user()
248: 	{
249: 		taxi::$check_auth_user_count++;
250: 		if (empty($_SESSION[UID])) return false;
251: 		$s = "SELECT 
252: 				`id_role`,
253: 				`id_user`,
254: 				`name`,
255: 				`family`,
```

#### `archive_17012026_1259/taxi/models/m_functions.php:4745`
```php
4740: 	}
4741: 
4742: 	 function auth_user($id_user)
4743: 	 {
4744: 		$_SESSION[UID] = $id_user;
4745: 		check_auth_user();
4746: 		if (empty($_SESSION[UID])) return false;
4747: 		$_SESSION['auth_time'] = time();
4748: 		$vfoc = md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))));
4749: 		setcookie("vfoc", $vfoc, 0, ROOT_URL);
4750: 		return openssl____encrypt(session_id() . "|" . $vfoc);
4751: 	 }
4752: 
4753: 	function upload_to_dropbox($content,$filename_upload,$id_dropbox_link,$mode = 'add')
```

#### `archive_17012026_1259/taxi/controllers/c_index.php:2`
```php
1: <?php
2: check_auth_user();
3: if (!CONFIG_FROM_PATH || isset($_GET['route']))
4: {
5: 	redirect(url(strlen($_SERVER['QUERY_STRING']) > 0 ? '?' . $_SERVER['QUERY_STRING'] : '',CONFIG_URL,0));	
6: }
7: ?>
8: <!DOCTYPE html>
9: <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7" lang="ru"> <![endif]-->  
10: <!--[if IE 7]><html class="no-js lt-ie9 lt-ie8" lang="ru"> <![endif]-->  
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:22`
```php
17: 
18: 		if (isset($_GET['par'][1]))
19: 		{
20: 			if ($_GET['par'][1] == 'register')
21: 			{
22: 				check_auth_user(); $API->id_role = taxi::$id_role;
23: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false);
24: 			}
25: 			elseif ($_GET['par'][1] == 'auth')
26: 			{
27: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
28: 
29: 			}
30: 			elseif ($_GET['par'][1] == 'logout')
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:36`
```php
31: 			{
32: 				$out = $API->logout();
33: 			}
34: 			elseif ($_GET['par'][1] == 'remind')
35: 			{
36: 				check_auth_user(); $API->id_role = taxi::$id_role;
37: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'');
38: 			}
39: 			elseif ($_GET['par'][1] == 'newpass')
40: 			{
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:41`
```php
36: 				check_auth_user(); $API->id_role = taxi::$id_role;
37: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'');
38: 			}
39: 			elseif ($_GET['par'][1] == 'newpass')
40: 			{
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
48: 				{
49: 					if (isset($_POST['data']))
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:46`
```php
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
48: 				{
49: 					if (isset($_POST['data']))
50: 					{
51: 						/*file_put_contents('req.log',$_POST['data']);
52: 	                    file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
53: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
54: 						/*file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:104`
```php
99: 					}
100: 				}
101: 			}
102: 			elseif ($_GET['par'][1] == 'car')
103: 			{
104: 				check_auth_user(); $API->id_role = taxi::$id_role;
105: 				if (empty($_GET['par'][3]))
106: 				{
107: 					if (isset($_POST['data']))
108: 					{
109: 						$out = $API->controlCar($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array());
110: 					}
111: 					else
112: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:128`
```php
123: 			{
124: 				$out = $API->updateCache(trim($_POST['hash']));
125: 			}
126: 			elseif ($_GET['par'][1] == 'drive')
127: 			{
128: 				check_auth_user(); $API->id_role = taxi::$id_role;
129: 				if (empty($_GET['par'][2]))
130: 				{
131: 					if (isset($_POST['data']))
132: 					{
133: 						$out = $API->createOrder($_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['payment_card'])?taxi::$data['payment_card']:array(),isset(taxi::$data['booking_comments'])?taxi::$data['booking_comments']:array(),isset(taxi::$data['services'])?taxi::$data['services']:array(),isset(taxi::$data['contact_classes'])?taxi::$data['contact_classes']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['unit_sets'])?taxi::$data['unit_sets']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array(),isset(taxi::$data_sc['schedule'])?taxi::$data_sc['schedule']:array());
134: 					}
135: 					else
136: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:211`
```php
206: 					}
207: 				}
208: 			}
209: 			elseif ($_GET['par'][1] == 'trip')
210: 			{
211: 				check_auth_user(); $API->id_role = taxi::$id_role;
212: 				if (empty($_GET['par'][2]))
213: 				{
214: 					if (isset($_POST['data']))
215: 					{
216: 						$out = $API->createTrip(empty($_POST['u_id'])?'':trim($_POST['u_id']),$_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),DEFAULT_PROFILE == 'stadium' ? array(taxi::$data_stt,taxi::$data_sc) : array());
217: 					}
218: 					else
219: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:300`
```php
295: 			}
296: 			elseif ($_GET['par'][1] == 'location')
297: 			{
298: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
299: 				{
300: 					check_auth_user(); $API->id_role = taxi::$id_role;
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:306`
```php
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:343`
```php
338: 			}
339: 			elseif ($_GET['par'][1] == 'data')
340: 			{
341: 				if (isset($_POST['data']))
342: 				{
343: 					check_auth_user(); $API->id_role = taxi::$id_role;
344: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
345: 				}
346: 				elseif (isset($_REQUEST['private']))
347: 				{
348: 					check_auth_user(); $API->id_role = taxi::$id_role;
349: 					$out = $API->selectDataPrivate(taxi::$data_private);
350: 				}
351: 				elseif (empty($_REQUEST['fields']))
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:348`
```php
343: 					check_auth_user(); $API->id_role = taxi::$id_role;
344: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
345: 				}
346: 				elseif (isset($_REQUEST['private']))
347: 				{
348: 					check_auth_user(); $API->id_role = taxi::$id_role;
349: 					$out = $API->selectDataPrivate(taxi::$data_private);
350: 				}
351: 				elseif (empty($_REQUEST['fields']))
352: 				{
353: 					$out = $API->selectData(isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,taxi::$version,taxi::$data);
354: 				}
355: 				else
356: 				{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:383`
```php
378: 					{
379: 						if (empty($_GET['par'][3]))
380: 						{
381: 							if (isset($_POST['file']))
382: 							{
383: 								check_auth_user(); $API->id_role = taxi::$id_role;
384: 								$out = $API->createDropboxFile($_POST['file']);
385: 							}
386: 						}
387: 						else
388: 						{
389: 							check_auth_user(); $API->id_role = taxi::$id_role;
390: 							$out = $API->selectDropboxFile(trim($_GET['par'][3]));
391: 						}
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:389`
```php
384: 								$out = $API->createDropboxFile($_POST['file']);
385: 							}
386: 						}
387: 						else
388: 						{
389: 							check_auth_user(); $API->id_role = taxi::$id_role;
390: 							$out = $API->selectDropboxFile(trim($_GET['par'][3]));
391: 						}
392: 					}
393: 					elseif ($_GET['par'][2] == 'update')
394: 					{
395: 						if (isset($_REQUEST['code']) && !(empty($_POST['hash'])))
396: 						{
397: 							$out = $API->updateDropbox(trim($_REQUEST['code']),trim($_POST['hash']));
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:404`
```php
399: 					}
400: 				}			
401: 			}
402: 			elseif ($_GET['par'][1] == 'cart')
403: 			{
404: 				check_auth_user(); $API->id_role = taxi::$id_role;
405: 				if (empty($_GET['par'][2]))
406: 				{
407: 					if(empty($_REQUEST['prod']))
408: 					{
409: 						$out = $API->selectCart(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
410: 					}
411: 					else
412: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:426`
```php
421: 					}
422: 				}
423: 			}
424: 			elseif ($_GET['par'][1] == 'cart_block')
425: 			{
426: 				check_auth_user(); $API->id_role = taxi::$id_role;
427: 				if (empty($_GET['par'][2]))
428: 				{
429: 					if(empty($_REQUEST['prod']))
430: 					{
431: 						$out = $API->selectCartBlock(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
432: 					}
433: 					else
434: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:452`
```php
447: 					}
448: 				}
449: 			}
450: 			elseif ($_GET['par'][1] == 'query')
451: 			{
452: 				check_auth_user(); $API->id_role = taxi::$id_role;
453: 				if (!empty($_GET['par'][2]))
454: 				{
455: 					if ($_GET['par'][2] == 'template')
456: 					{
457: 						$out = $API->queryTemplate(isset($_POST['data'])?$_POST['data']:'',empty($_GET['par'][3])?'':trim($_GET['par'][3]),isset(taxi::$data_private['sql_templates'])?taxi::$data_private['sql_templates']:array());
458: 					}
459: 					else
460: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:467`
```php
462: 					}
463: 				}
464: 			}
465: 			elseif ($_GET['par'][1] == 'schedule')
466: 			{
467: 				check_auth_user(); $API->id_role = taxi::$id_role;
468: 				if (!empty($_GET['par'][2]) && $_GET['par'][2] == 'ticket')
469: 				{
470: 					$out = $API->selectTicket(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'');
471: 				}
472: 			}
473: 			elseif ($_GET['par'][1] == 'email')
474: 			{
475: 				check_auth_user(); $API->id_role = taxi::$id_role;
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:475`
```php
470: 					$out = $API->selectTicket(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'');
471: 				}
472: 			}
473: 			elseif ($_GET['par'][1] == 'email')
474: 			{
475: 				check_auth_user(); $API->id_role = taxi::$id_role;
476: 				if (!empty($_GET['par'][2]))
477: 				{
478: 					if ($_GET['par'][2] == 'verification')
479: 					{
480: 						if ($_GET['par'][3] == 'start')
481: 						{
482: 							$out = $API->startEmailVerification();
483: 						}
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:493`
```php
488: 					}
489: 				}
490: 			}
491: 			elseif ($_GET['par'][1] == 'script')
492: 			{
493: 				check_auth_user(); $API->id_role = taxi::$id_role;
494: 				if (!empty($_GET['par'][2]))
495: 				{
496: 					if ($_GET['par'][2] == 'template')
497: 					{
498: 						$out = $API->includeTemplate(empty($_GET['par'][3])?'':trim($_GET['par'][3]),empty($_REQUEST['is_var'])?false:true,isset(taxi::$data_private['script_templates'])?taxi::$data_private['script_templates']:array());
499: 					}
500: 				}
501: 			}
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:520`
```php
515: 			$_REQUEST['au'] = trim($_REQUEST['au']);
516: 			if ($_REQUEST['au'] === '')
517: 			{
518: 				if (empty($out['auth_user']))
519: 				{
520: 					if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
521: 					$out['auth_user'] = array(
522: 											'u_id' => $_SESSION[UID],
523: 											'u_name' => $_SESSION['name'],
524: 											'u_family' => $_SESSION['family'],
525: 											'u_middle' => $_SESSION['middle'],
526: 											'u_email' => $_SESSION['email'],
527: 											'u_phone' => $_SESSION['phone'],
528: 											'u_role' => $_SESSION['id_role'],
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:549`
```php
544: 				{
545: 					if (empty($out['auth_user']) || !isset($out['auth_user']['u_description']))
546: 					{
547: 						if (empty($out['data']['user'][$_SESSION[UID]]))
548: 						{
549: 							if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
550: 							$out['auth_user'] = $API->selectUser($_SESSION[UID]);						
551: 							$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
552: 						}
553: 						else
554: 						{
555: 							$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
556: 						}
557: 						if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
```

#### `archive_17012026_1259/taxi/controllers/c_api_test.php:581`
```php
576: 				}
577: 				elseif ($_REQUEST['au'] === 'f')
578: 				{
579: 					if (empty($out['data']['user'][$_SESSION[UID]]))
580: 					{
581: 						if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
582: 						$out['auth_user'] = $API->selectUser($_SESSION[UID]);
583: 						$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
584: 					}
585: 					else
586: 					{
587: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
588: 					}
589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:26`
```php
21: 
22: 		if (isset($_GET['par'][1]))
23: 		{
24: 			if ($_GET['par'][1] == 'register')
25: 			{
26: 				check_auth_user(); $API->id_role = taxi::$id_role;
27: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false,isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
28: 			}
29: 			elseif ($_GET['par'][1] == 'auth')
30: 			{
31: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
32: 
33: 			}
34: 			elseif ($_GET['par'][1] == 'logout')
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:40`
```php
35: 			{
36: 				$out = $API->logout();
37: 			}
38: 			elseif ($_GET['par'][1] == 'remind')
39: 			{
40: 				check_auth_user(); $API->id_role = taxi::$id_role;
41: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'');
42: 			}
43: 			elseif ($_GET['par'][1] == 'newpass')
44: 			{
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:45`
```php
40: 				check_auth_user(); $API->id_role = taxi::$id_role;
41: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'');
42: 			}
43: 			elseif ($_GET['par'][1] == 'newpass')
44: 			{
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
52: 				{
53: 					if (isset($_POST['data']))
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:50`
```php
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
52: 				{
53: 					if (isset($_POST['data']))
54: 					{
55: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array(),isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
56: 					}
57: 					else
58: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:112`
```php
107: 					}
108: 				}
109: 			}
110: 			elseif ($_GET['par'][1] == 'car')
111: 			{
112: 				check_auth_user(); $API->id_role = taxi::$id_role;
113: 				if (empty($_GET['par'][3]))
114: 				{
115: 					if (isset($_POST['data']))
116: 					{
117: 						$out = $API->controlCar($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array());
118: 					}
119: 					else
120: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:136`
```php
131: 			{
132: 				$out = $API->updateCache(trim($_POST['hash']));
133: 			}
134: 			elseif ($_GET['par'][1] == 'drive')
135: 			{
136: 				check_auth_user(); $API->id_role = taxi::$id_role;
137: 				if (empty($_GET['par'][2]))
138: 				{
139: 					if (isset($_POST['data']))
140: 					{
141: 						$out = $API->createOrder($_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['payment_card'])?taxi::$data['payment_card']:array(),isset(taxi::$data['booking_comments'])?taxi::$data['booking_comments']:array(),isset(taxi::$data['services'])?taxi::$data['services']:array(),isset(taxi::$data['contact_classes'])?taxi::$data['contact_classes']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['unit_sets'])?taxi::$data['unit_sets']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array(),isset(taxi::$data_sc['schedule'])?taxi::$data_sc['schedule']:array(),isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array(),empty(taxi::$data['site_constants']['stripe_seat_title_template'])?'':taxi::$data['site_constants']['stripe_seat_title_template']['value'],empty(taxi::$data['site_constants']['stripe_request_duration'])?0:taxi::$data['site_constants']['stripe_request_duration']['value'],isset(taxi::$data_private['aggregators'])?taxi::$data_private['aggregators']:array());
142: 					}
143: 					else
144: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:219`
```php
214: 					}
215: 				}
216: 			}
217: 			elseif ($_GET['par'][1] == 'trip')
218: 			{
219: 				check_auth_user(); $API->id_role = taxi::$id_role;
220: 				if (empty($_GET['par'][2]))
221: 				{
222: 					if (isset($_POST['data']))
223: 					{
224: 						$out = $API->createTrip(empty($_POST['u_id'])?'':trim($_POST['u_id']),$_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),DEFAULT_PROFILE == 'stadium' ? array(taxi::$data,taxi::$data_stt,taxi::$data_sc) : array());
225: 					}
226: 					else
227: 					{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:308`
```php
303: 			}
304: 			elseif ($_GET['par'][1] == 'location')
305: 			{
306: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
307: 				{
308: 					check_auth_user(); $API->id_role = taxi::$id_role;
309: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
310: 				}
311: 			}
312: 			elseif ($_GET['par'][1] == 'token')
313: 			{
314: 				check_auth_user(); $API->id_role = taxi::$id_role;
315: 				if (empty($_GET['par'][3]))
316: 				{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:314`
```php
309: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
310: 				}
311: 			}
312: 			elseif ($_GET['par'][1] == 'token')
313: 			{
314: 				check_auth_user(); $API->id_role = taxi::$id_role;
315: 				if (empty($_GET['par'][3]))
316: 				{
317: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
318: 
319: 				}
320: 			}
321: 			elseif ($_GET['par'][1] == 'mail')
322: 			{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:351`
```php
346: 			}
347: 			elseif ($_GET['par'][1] == 'data')
348: 			{
349: 				if (isset($_POST['data']))
350: 				{
351: 					check_auth_user(); $API->id_role = taxi::$id_role;
352: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
353: 				}
354: 				elseif (isset($_REQUEST['private']))
355: 				{
356: 					check_auth_user(); $API->id_role = taxi::$id_role;
357: 					$out = $API->selectDataPrivate(taxi::$data_private,isset($_REQUEST['json_like'])?$_REQUEST['json_like']:'');
358: 				}
359: 				elseif (empty($_REQUEST['fields']))
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:356`
```php
351: 					check_auth_user(); $API->id_role = taxi::$id_role;
352: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
353: 				}
354: 				elseif (isset($_REQUEST['private']))
355: 				{
356: 					check_auth_user(); $API->id_role = taxi::$id_role;
357: 					$out = $API->selectDataPrivate(taxi::$data_private,isset($_REQUEST['json_like'])?$_REQUEST['json_like']:'');
358: 				}
359: 				elseif (empty($_REQUEST['fields']))
360: 				{
361:                     $out = $API->selectData(isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,isset($_REQUEST['json_like'])?$_REQUEST['json_like']:'',taxi::$version,taxi::$data);
362: 				}
363: 				else
364: 				{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:391`
```php
386: 					{
387: 						if (empty($_GET['par'][3]))
388: 						{
389: 							if (isset($_POST['file']))
390: 							{
391: 								check_auth_user(); $API->id_role = taxi::$id_role;
392: 								$out = $API->createDropboxFile($_POST['file']);
393: 							}
394: 						}
395: 						else
396: 						{
397: 							if (empty($_GET['par'][4]))
398: 							{
399: 								check_auth_user(); $API->id_role = taxi::$id_role;
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:399`
```php
394: 						}
395: 						else
396: 						{
397: 							if (empty($_GET['par'][4]))
398: 							{
399: 								check_auth_user(); $API->id_role = taxi::$id_role;
400: 								$out = $API->selectDropboxFile(trim($_GET['par'][3]));
401: 							}
402: 							elseif ($_GET['par'][4] == 'del')
403: 							{
404: 								check_auth_user(); $API->id_role = taxi::$id_role;
405: 								$out = $API->deleteDropboxFile(trim($_GET['par'][3]));
406: 							}
407: 							elseif ($_GET['par'][4] == 'select')
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:404`
```php
399: 								check_auth_user(); $API->id_role = taxi::$id_role;
400: 								$out = $API->selectDropboxFile(trim($_GET['par'][3]));
401: 							}
402: 							elseif ($_GET['par'][4] == 'del')
403: 							{
404: 								check_auth_user(); $API->id_role = taxi::$id_role;
405: 								$out = $API->deleteDropboxFile(trim($_GET['par'][3]));
406: 							}
407: 							elseif ($_GET['par'][4] == 'select')
408: 							{
409: 								check_auth_user(); $API->id_role = taxi::$id_role;
410: 								$out = $API->getDropboxFileData(trim($_GET['par'][3]),isset($_REQUEST['private'])?trim($_REQUEST['private']):'',isset($_REQUEST['deleted'])?trim($_REQUEST['deleted']):'');
411: 							}
412: 						}
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:409`
```php
404: 								check_auth_user(); $API->id_role = taxi::$id_role;
405: 								$out = $API->deleteDropboxFile(trim($_GET['par'][3]));
406: 							}
407: 							elseif ($_GET['par'][4] == 'select')
408: 							{
409: 								check_auth_user(); $API->id_role = taxi::$id_role;
410: 								$out = $API->getDropboxFileData(trim($_GET['par'][3]),isset($_REQUEST['private'])?trim($_REQUEST['private']):'',isset($_REQUEST['deleted'])?trim($_REQUEST['deleted']):'');
411: 							}
412: 						}
413: 					}
414: 					elseif ($_GET['par'][2] == 'update')
415: 					{
416: 						if (isset($_REQUEST['code']) && !(empty($_POST['hash'])))
417: 						{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:425`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:453`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:479`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:488`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:494`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:509`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:527`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:538`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:543`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:554`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:593`
```php
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
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:650`
```php
645: 				{
646: 					if ($_GET['par'][2] == 'check')
647: 					{
648: 						if (empty($_GET['par'][3]))
649: 						{
650: 							check_auth_user(); $API->id_role = taxi::$id_role;
651: 							$out = $API->checkTicket(isset($_REQUEST['code'])?trim($_REQUEST['code']):'');
652: 						}
653: 						else
654: 						{
655: 							if ($_GET['par'][3] == 'log')
656: 							{
657: 								check_auth_user(); $API->id_role = taxi::$id_role;
658: 								$out = $API->selectCheckTicketLog(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'',isset($_REQUEST['seat'])?trim($_REQUEST['seat']):'',isset($_REQUEST['u_id'])?trim($_REQUEST['u_id']):'',isset($_REQUEST['pass'])?trim($_REQUEST['pass']):NULL);
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:657`
```php
652: 						}
653: 						else
654: 						{
655: 							if ($_GET['par'][3] == 'log')
656: 							{
657: 								check_auth_user(); $API->id_role = taxi::$id_role;
658: 								$out = $API->selectCheckTicketLog(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'',isset($_REQUEST['seat'])?trim($_REQUEST['seat']):'',isset($_REQUEST['u_id'])?trim($_REQUEST['u_id']):'',isset($_REQUEST['pass'])?trim($_REQUEST['pass']):NULL);
659: 							}
660: 						}
661: 
662: 					}				
663: 				}
664: 			}
665: 			elseif ($_GET['par'][1] == 'promocode')
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:671`
```php
666: 			{
667: 				if (!empty($_GET['par'][2]))
668: 				{
669: 					if ($_GET['par'][2] == 'check')
670: 					{
671: 						check_auth_user(); $API->id_role = taxi::$id_role;
672: 						$out = $API->selectPromocodeByValue(isset($_REQUEST['value'])?trim($_REQUEST['value']):'',isset($_REQUEST['count'])?trim($_REQUEST['count']):NULL);
673: 					}				
674: 				}
675: 			}
676: 			elseif ($_GET['par'][1] == 'payment')
677: 			{
678: 				if (!empty($_GET['par'][2]))
679: 				{
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:680`
```php
675: 			}
676: 			elseif ($_GET['par'][1] == 'payment')
677: 			{
678: 				if (!empty($_GET['par'][2]))
679: 				{
680: 					check_auth_user(); $API->id_role = taxi::$id_role;
681: 					if ($_GET['par'][2] == 'create')
682: 					{
683: 						$out = $API->createPayment(isset($_POST['data'])?$_POST['data']:'',isset($_REQUEST['appUrl'])?trim($_REQUEST['appUrl']):'',isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array());
684: 					}
685: 					elseif ($_GET['par'][2] == 'get')
686: 					{
687: 						$out = $API->selectPayment(isset($_REQUEST['p_id'])?trim($_REQUEST['p_id']):NULL,isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array());
688: 					}
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:695`
```php
690: 			}
691: 			elseif ($_GET['par'][1] == 'subscription')
692: 			{
693: 				if (!empty($_GET['par'][2]))
694: 				{
695: 					check_auth_user(); $API->id_role = taxi::$id_role;
696: 					if ($_GET['par'][2] == 'create')
697: 					{
698: 						$out = $API->createSubscription(isset($_POST['data'])?$_POST['data']:'');
699: 					}
700: 					elseif ($_GET['par'][2] == 'get')
701: 					{
702: 						$out = $API->selectSubscription(isset($_REQUEST['subs_id'])?trim($_REQUEST['subs_id']):NULL,isset($_REQUEST['u_id'])?trim($_REQUEST['u_id']):NULL);
703: 					}
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:710`
```php
705: 			}
706: 			elseif ($_GET['par'][1] == 'account')
707: 			{
708: 				if (!empty($_GET['par'][2]))
709: 				{
710: 					check_auth_user(); $API->id_role = taxi::$id_role;
711: 					if ($_GET['par'][2] == 'deposit')
712: 					{
713: 						$out = $API->depositCurrencyAccount(isset($_POST['data'])?$_POST['data']:'',isset($_REQUEST['appUrl'])?trim($_REQUEST['appUrl']):'',isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['site_constants']['account_currency_list'])? array_flip(explode(',',taxi::$data['site_constants']['account_currency_list']['value'])):array());
714: 					}
715: 					elseif ($_GET['par'][2] == 'withdraw')
716: 					{
717: 						$out = $API->withdrawCurrencyAccount(isset($_POST['data'])?$_POST['data']:'',isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['site_constants']['account_currency_list'])? array_flip(explode(',',taxi::$data['site_constants']['account_currency_list']['value'])):array());
718: 					}
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:737`
```php
732: 			}
733: 			elseif ($_GET['par'][1] == 'deal')
734: 			{
735: 				if (!empty($_GET['par'][2]))
736: 				{
737: 					check_auth_user(); $API->id_role = taxi::$id_role;
738: 					if ($_GET['par'][2] == 'create')
739: 					{
740: 						$out = $API->createDeal(isset($_POST['data'])?$_POST['data']:'',isset(taxi::$data['site_constants']['account_currency_list'])? array_flip(explode(',',taxi::$data['site_constants']['account_currency_list']['value'])):array());
741: 					}
742: 					elseif ($_GET['par'][2] == 'complete')
743: 					{
744: 						$out = $API->completeDeal(isset($_REQUEST['d_id'])?trim($_REQUEST['d_id']):NULL);
745: 					}
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:774`
```php
769: 			$_REQUEST['au'] = trim($_REQUEST['au']);
770: 			if ($_REQUEST['au'] === '')
771: 			{
772: 				if (empty($out['auth_user']))
773: 				{
774: 					if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
775: 					$out['auth_user'] = array(
776: 											'u_id' => $_SESSION[UID],
777: 											'u_name' => $_SESSION['name'],
778: 											'u_family' => $_SESSION['family'],
779: 											'u_middle' => $_SESSION['middle'],
780: 											'u_email' => $_SESSION['email'],
781: 											'u_phone' => $_SESSION['phone'],
782: 											'u_role' => $_SESSION['id_role'],
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:803`
```php
798: 				{
799: 					if (empty($out['auth_user']) || !isset($out['auth_user']['u_description']))
800: 					{
801: 						if (empty($out['data']['user'][$_SESSION[UID]]))
802: 						{
803: 							if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
804: 							$out['auth_user'] = $API->selectUser($_SESSION[UID]);						
805: 							$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
806: 						}
807: 						else
808: 						{
809: 							$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
810: 						}
811: 						if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
```

#### `archive_17012026_1259/taxi/controllers/c_api.php:841`
```php
836: 				}
837: 				elseif ($_REQUEST['au'] === 'f')
838: 				{
839: 					if (empty($out['data']['user'][$_SESSION[UID]]))
840: 					{
841: 						if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
842: 						$out['auth_user'] = $API->selectUser($_SESSION[UID]);
843: 						$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
844: 					}
845: 					else
846: 					{
847: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
848: 					}
849: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
```

## 3. Что подтверждено

В source corpus `check_auth_user()` выступает как общий authentication/session gate и вызывается из API paths. Это подтверждает общий механизм:

```text
Request
  ↓
check_auth_user()
  ↓
authenticated API context
```

Но `check_auth_user()` сам по себе не является доказанным универсальным Role × Operation authorization helper.

Role-specific gates продолжают выполняться отдельно:

```text
check_auth_user()
      ↓
runtime id_role
      ↓
operation-local role check
      ↓
ALLOW / REJECT
```

## 4. Архитектурное различие

### Authentication helper

```text
check_auth_user()
    → establishes authenticated request/session context```

### Authorization gates

```text
query_rolesrole comparisonsverification statecustom hash / role gates    → operation-specific access decisions```

Следовательно, объединять всё в один `AuthorizationHelper` пока нельзя.

## 5. Semantic Graph update

Допустимые Claims:

```text
API Request
    ──USES──>
Authentication Gate

Authentication Gate
    ──IMPLEMENTED_BY──>
check_auth_user()

Protected Operation
    ──HAS_AUTHORIZATION_GATE──>
operation-local role check```

Не вводим пока:

```text
All Authorization
    ──IMPLEMENTED_BY──>
check_auth_user()```

Это было бы семантическим overreach.

## 6. Что происходит с RP-19

RP-19 обнаружил общий **pattern**, но RP-20 показывает, что общий pattern не равен общему helper.

```text
common pattern
    ≠
common implementation```

Это важный результат для методологии.

## 7. Текущая модель Authorization

```text
Request
   ↓
Authentication
   ↓
check_auth_user()
   ↓
Authenticated Context
   ↓
Operation
   ↓
┌──────────────────────────────┐
│ operation-local auth gates   │
│                              │
│ role set                     │
│ role comparison              │
│ verification precondition    │
│ statement-specific gate      │
└──────────────────────────────┘
   ↓
ALLOW / REJECT
```

## 8. Gap Report

```text
G-20-01  enumerate all operation-local authorization gates   OPEN
G-20-02  identify any second-level shared authorization helper OPEN
G-20-03  complete Role × Operation matrix                    OPEN
G-20-04  map frontend-consumed authorization capabilities    OPEN
```

## 9. MCR

`MCR = NO CHANGE`.

Не создаётся новый фундаментальный graph entity. Разделение Authentication и Authorization фиксируется как relation/type distinction.

## 10. Следующий шаг

Следующий проход должен уже не искать helper names, а взять **один конкретный protected operation кроме `/query`**, полностью пройти его:

```text
request
 ↓
authentication
 ↓
authorization gate(s)
 ↓
preconditions
 ↓
decision
 ↓
implementation/result```

Это даст первую полноценную строку Role × Operation × Preconditions × Evidence вне `/query`.