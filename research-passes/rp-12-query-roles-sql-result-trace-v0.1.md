# Backend Semantic Graph — Research Pass 12
# `query_roles` SQL / Result Trace v0.1

**Статус:** PROVISIONAL / PARTIALLY ANSWERED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-11 `query_roles` Result Boundary v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Цель

Дойти от конкретного `query_roles` consumer до SQL/query predicate и определить фактический semantic effect. Не считать роль authorization, если код только ограничивает набор данных.

## 2. Найденные consumer functions
- `archive_17012026_1259/taxi/cache/data.php` — `global scope` — `query_roles` около line 13845
- `archive_17012026_1259/taxi/controllers/c_api.php` — `global scope` — `query_roles` около line 488
- `archive_17012026_1259/taxi/controllers/c_index.php` — `global scope` — `query_roles` около line 3364
- `archive_17012026_1259/taxi/controllers/c_index.php` — `get_fields` — `query_roles` около line 9269
- `archive_17012026_1259/taxi/models/api.php` — `queryString` — `query_roles` около line 17898

## 3. Доказательная трасса
### `global scope` — `archive_17012026_1259/taxi/cache/data.php`
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
13966:         "e_alarm_power_cut_off",
13967:         "e_alarm_speeding_end",
13968:         "e_alarm_shake"
13969:     ]
13970: }',
13971:     ),
13972:     'sms_code_ip_attempts_count' => 
13973:     array (
13974:       'group' => '20',
13975:       'ru' => 'Максимальное количество доступных смс с одного ip в течении промежутка времени, определяемого константой sms_code_ip_attempts_count_interval.',
13976:       'en' => NULL,
13977:       'ar' => NULL,
13978:       'fr' => NULL,
13979:       'es' => NULL,
13980:       'about_ru' => 'Любое значение, кроме числа больше нуля, означает, что ограничения на количество отправляемых сообщений нет.',
13981:       'about_en' => '',
13982:       'about_ar' => '',
13983:       'about_fr' => '',
13984:       'about_es' => '',
13985:       'value' => '0',
13986:     ),
13987:     'sms_code_ip_attempts_count_interval' => 
13988:     array (
13989:       'group' => '20',
13990:       'ru' => 'Интервал времени, в течении которого считается количество доступных смс с одного ip.',
13991:       'en' => NULL,
13992:       'ar' => NULL,
13993:       'fr' => NULL,
13994:       'es' => NULL,
13995:       'about_ru' => 'Значение в секундах. Любое значение, кроме числа больше нуля, означает, что считается за все время.',
13996:       'about_en' => '',
13997:       'about_ar' => '',
13998:       'about_fr' => '',
13999:       'about_es' => '',
14000:       'value' => '86400',
14001:     ),
14002:     'sms_code_ip_attempts_interval' => 
14003:     array (
14004:       'group' => '20',
14005:       'ru' => 'Минимальный интервал между смс с одного ip.',
14006:       'en' => NULL,
14007:       'ar' => NULL,
14008:       'fr' => NULL,
14009:       'es' => NULL,
14010:       'about_ru' => 'Значение в секундах. Любое значение, кроме числа больше нуля, означает, что ограничений на частоту смс с одного ip нет.',
14011:       'about_en' => '',
14012:       'about_ar' => '',
14013:       'about_fr' => '',
14014:       'about_es' => '',
14015:       'value' => '0',
14016:     ),
14017:     'sms_code_attempts_count' => 
14018:     array (
14019:       'group' => '20',
14020:       'ru' => 'Максимальное количество доступных смс для одного контакта одного вида логина в течении промежутка времени, определяемого константой sms_code_attempts_count_interval.',
14021:       'en' => NULL,
14022:       'ar' => NULL,
14023:       'fr' => NULL,
14024:       'es' => NULL,
14025:       'about_ru' => 'Любое значение, кроме числа больше нуля, означает, что ограничения на количество отправляемых сообщений нет.',
14026:       'about_en' => '',
14027:       'about_ar' => '',
14028:       'about_fr' => '',
14029:       'about_es' => '',
14030:       'value' => '0',
14031:     ),
14032:     'sms_code_attempts_count_interval' => 
14033:     array (
14034:       'group' => '20',
14035:       'ru' => 'Интервал времени, в течении которого считается количество доступных смс для одного контакта одного вида логина.',
14036:       'en' => NULL,
14037:       'ar' => NULL,
14038:       'fr' => NULL,
14039:       'es' => NULL,
14040:       'about_ru' => 'Значение в секундах. Любое значение, кроме числа больше нуля, означает, что считается за все время.',
14041:       'about_en' => '',
14042:       'about_ar' => '',
14043:       'about_fr' => '',
14044:       'about_es' => '',
14045:       'value' => '86400',
14046:     ),
14047:     'sms_code_on' => 
14048:     array (
14049:       'group' => '20',
14050:       'ru' => 'Разрешить ли отправку смс.',
14051:       'en' => NULL,
14052:       'ar' => NULL,
14053:       'fr' => NULL,
14054:       'es' => NULL,
14055:       'about_ru' => 'Непустое значение включает отправку.',
14056:       'about_en' => '',
14057:       'about_ar' => '',
14058:       'about_fr' => '',
14059:       'about_es' => '',
14060:       'value' => '1',
14061:     ),
14062:     'commission_deposit' => 
14063:     array (
14064:       'group' => '21',
14065:       'ru' => 'Комиссия за пополнение в процентах.',
14066:       'en' => NULL,
14067:       'ar' => NULL,
14068:       'fr' => NULL,
14069:       'es' => NULL,
14070:       'about_ru' => '',
14071:       'about_en' => '',
14072:       'about_ar' => '',
14073:       'about_fr' => '',
14074:       'about_es' => '',
14075:       'value' => '0',
14076:     ),
14077:     'commission_withdraw' => 
14078:     array (
14079:       'group' => '21',
14080:       'ru' => 'Комиссия за вывод в процентах.',
14081:       'en' => NULL,
14082:       'ar' => NULL,
14083:       'fr' => NULL,
14084:       'es' => NULL,
```

### `global scope` — `archive_17012026_1259/taxi/controllers/c_api.php`
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
609: 						{
610: 							if ($_GET['par'][3] == 'add')
611: 							{
612: 								$out = $API->addTaskLog(isset($_POST['data'])?$_POST['data']:'');
613: 							}
614: 							elseif ($_GET['par'][3] == 'get')
615: 							{
616: 								$out = $API->getTaskLog(isset($_REQUEST['tl_id'])?trim($_REQUEST['tl_id']):NULL);
617: 							}
618: 						}
619: 					}
620: 					elseif ($_GET['par'][2] == 'status')
621: 					{
622: 						$out = $API->setTaskStatus(isset($_REQUEST['tl_id'])?trim($_REQUEST['tl_id']):NULL,isset($_REQUEST['from_status'])?trim($_REQUEST['from_status']):NULL,isset($_REQUEST['to_status'])?trim($_REQUEST['to_status']):NULL);
623: 					}
624: 				}
625: 			}
626: 			elseif ($_GET['par'][1] == 'convert')
627: 			{
628: 				if (!empty($_GET['par'][2]))
629: 				{
630: 					if ($_GET['par'][2] == 'html')
631: 					{
632: 						if (!empty($_GET['par'][3]))
633: 						{
634: 							if ($_GET['par'][3] == 'pdf')
635: 							{
636: 								$out = $API->htmlToPdf(isset($_POST['data'])?$_POST['data']:'');
637: 							}
638: 						}
639: 					}
640: 				}
641: 			}
642: 			elseif ($_GET['par'][1] == 'ticket')
643: 			{
644: 				if (!empty($_GET['par'][2]))
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
659: 							}
660: 						}
661: 
662: 					}				
663: 				}
664: 			}
665: 			elseif ($_GET['par'][1] == 'promocode')
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
680: 					check_auth_user(); $API->id_role = taxi::$id_role;
681: 					if ($_GET['par'][2] == 'create')
682: 					{
683: 						$out = $API->createPayment(isset($_POST['data'])?$_POST['data']:'',isset($_REQUEST['appUrl'])?trim($_REQUEST['appUrl']):'',isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array());
684: 					}
685: 					elseif ($_GET['par'][2] == 'get')
686: 					{
687: 						$out = $API->selectPayment(isset($_REQUEST['p_id'])?trim($_REQUEST['p_id']):NULL,isset(taxi::$data['payment_services'])?taxi::$data['payment_services']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array());
688: 					}
689: 				}
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
704: 				}
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
719: 					elseif ($_GET['par'][2] == 'transfer')
720: 					{
721: 						$out = $API->transferCurrencyAccount(isset($_POST['data'])?$_POST['data']:'',isset(taxi::$data['site_constants']['account_currency_list'])? array_flip(explode(',',taxi::$data['site_constants']['account_currency_list']['value'])):array());
722: 					}
723: 					elseif ($_GET['par'][2] == 'get')
724: 					{
725: 						$out = $API->selectCurrencyAccount(isset($_REQUEST['a_id'])?trim($_REQUEST['a_id']):NULL,isset($_REQUEST['u_id'])?trim($_REQUEST['u_id']):NULL);
726: 					}
727: 					elseif ($_GET['par'][2] == 'transaction')
```

### `global scope` — `archive_17012026_1259/taxi/controllers/c_index.php`
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
3453: 				<label class="no_border"><span>идентификатор шаблона (data.sql_templates)</span><input id="query_template_te_id" type="text"></label>
3454: 				<label><span>данные для подстановки</span><input data-name="data" name="data" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3455: 				<label class="exclude"><span>название sql переменной, значение которой будет возвращено</span><input data-name="var" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3456: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3457: 			</form>
3458: 		</fieldset>
3459: 		<pre>
3460: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/write/				POST
3461: 			Загрузка и удаление билета на сайт
3462: 				Доступно только для авторизованного пользователя.
3463: 				Доступно только пользователю, создавшему t_id или администратору.
3464: 				Параметры запроса:				
3465: 					data		JSON.stringify([{
3466: 										"base64": 		файл, кодированный в base64 строку
3467: 										"seat":			место билета
3468: 										"extW.Dot":		расширение файла, включая точку
3469: 									},...])
3470: 				Если base64="", то билет удаляется.
3471: 				Ответ сервера:
3472: 				{'code':'200','status':'success'}}</pre>				
3473: 		<pre>
3474: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/read/				GET
3475: 			Получение списка загоуженных билетов
3476: 			Доступно только для авторизованного пользователя.
3477: 				Доступно только пользователю, создавшему t_id или администратору.
3478: 			Ответ сервера:
3479: 			{'code':'200','status':'success','data:{'seats':[...]}}</pre>
3480: 		<pre>
3481: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/read/				GET
3482: 			Получение билета
3483: 			Доступно только для авторизованного пользователя.
3484: 			Доступно пользователю, или создавшему t_id или купишему seat, а также администратору.
3485: 			Параметры запроса:	
3486: 				seat 		номер места билета, GET или POST
3487: 				pdf			если указан, то отдается pdf
3488: 			Ответ сервера: файл</pre>
3489: 		<pre>
3490: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/send/buyer/email	POST
3491: 			Отправка сообщения покупателю
3492: 				Доступно только для авторизованного пользователя.
3493: 				Доступно только пользователю, создавшему t_id.
3494: 				Параметры запроса:	
3495: 					seat				номер места билета, GET или POST
3496: 					subject				заголовок сообщения или 'template', POST
3497: 					body				содержание или 'template', POST	
3498: 					file				JSON.stringify([
3499: 															{
3500: 																"base64": 	файл, кодированный в base64 строку или 'uploaded' или 'pdf'
3501: 																"name":		имя файла с расширением
3502: 															},
3503: 															...
3504: 													   ]), POST	
3505: 					u_id				идентификатор юзера, которому отправляется, GET или POST (требует ignore_order, не для resend)
3506: 					email				емейл, на который отправляется, POST
3507: 					u_name				имя покупателя, POST (не для resend)	
3508: 					ignore_order		если указан, покупка и оплата не проверяется и цена не берется из заказа, GET или POST
3509: 					price				цена, GET или POST (не для resend)
3510: 					resend				если указан, послать снова, GET или POST
3511: 				Ответ сервера:
3512: 				{'code':'200','status':'success'}}</pre>		
3513: 		<fieldset class="form"><legend title="Загружает или удаляет билет.">Загрузка и удаление билета</legend>
3514: 			<form id="trip_ticket_write" class="complex" action="api/v1/trip/get/" data-action="api/v1/trip/get/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3515: 				<label class="no_border"><span>идентификатор рейса</span><input id="trip_ticket_write_t_id" type="text"></label>
3516: 				<label class="json_key1"><span>файл билета 1, кодированный в base64 строку</span><input data-name="base64" type="text" onclick="file_to_dataurl(this,'fname')"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3517: 				<label class="json_key1"><span>место билета 1</span><input data-name="seat" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3518: 				<label class="json_key1 exclude"><span>расширение файла билета 1, включая точку</span><input data-name="extW.Dot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3519: 				<label class="json_key2 exclude"><span>файл билета 2, кодированный в base64 строку</span><input data-name="base64" type="text" onclick="file_to_dataurl(this,'fname')"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3520: 				<label class="json_key2 exclude"><span>место билета 2</span><input data-name="seat" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3521: 				<label class="json_key2 exclude"><span>расширение файла билета 2, включая точку</span><input data-name="extW.Dot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3522: 				<input id="trip_ticket_write_data" data-name="data" type="hidden">
3523: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3524: 			</form>
3525: 		</fieldset>
3526: 		<fieldset class="form"><legend title="Читает билет или список.">Получение билета</legend>
3527: 			<form id="trip_ticket_read" class="complex" action="api/v1/trip/get/" data-action="api/v1/trip/get/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3528: 				<label class="no_border"><span>идентификатор рейса</span><input id="trip_ticket_read_t_id" type="text"></label>
3529: 				<label><span>место билета</span><input data-name="seat" name="seat" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3530: 				<label><span>вывод pdf</span><input class="checkbox" name="pdf" type="checkbox" value=""></label>
3531: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3532: 			</form>
3533: 		</fieldset>
3534: 
3535: 		<fieldset class="form"><legend title="Отправка сообщения покупателю.">Отправляет сообщение покупателю</legend>
3536: 			<form id="trip_ticket_send" class="complex" action="api/v1/trip/get/" data-action="api/v1/trip/get/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3537: 				<label class="no_border"><span>идентификатор рейса</span><input id="trip_ticket_send_t_id" type="text"></label>
3538: 				<label><span>место билета</span><input data-name="seat" name="seat" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>			
3539: 				<label><span>заголовок сообщения</span><input data-name="subject" name="subject" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3540: 				<label><span>содержание сообщения</span><textarea data-name="body" name="body"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3541: 				<label class="json_key1"><span>файл 1, кодированный в base64 строку</span><input data-name="base64" type="text" onclick="file_to_dataurl(this,'fname')"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3542: 				<label class="json_key1 exclude"><span>имя файла 1</span><input id="trip_ticket_send_name1" data-name="name" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3543: 				<label class="json_key2 exclude"><span>файл 2, кодированный в base64 строку</span><input data-name="base64" type="text" onclick="file_to_dataurl(this,'fname')"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3544: 				<label class="json_key2 exclude"><span>имя файла 2</span><input id="trip_ticket_send_name2" data-name="name" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
3545: 				<label class="exclude"><span>идентификатор юзера получателя</span><textarea data-name="u_id"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3546: 				<label class="exclude"><span>емейл получателя</span><textarea data-name="email"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3547: 				<label class="exclude"><span>имя получателя</span><textarea data-name="u_name"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3548: 				<label><span>игнорировать заказ</span><input class="checkbox" name="ignore_order" type="checkbox" value=""></label>
3549: 				<label class="exclude"><span>цена</span><textarea data-name="price"></textarea> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3550: 				<label><span>отправить повторно</span><input class="checkbox" name="resend" type="checkbox" value=""></label>
3551: 				<input id="trip_ticket_send_data" data-name="file" type="hidden">
3552: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3553: 			</form>
3554: 		</fieldset>
3555: 
3556: 		<pre>
3557: 		https://ibronevik.ru/taxi/api/v1/cart_block								GET
3558: 			Корзина пользователя для оповещения и предзаказа.		
3559: 				Доступно только для авторизованного пользователя.
3560: 				GET или POST параметр filter:
3561: 					all			выводит все, доступен админу и продавцу
3562: 					trip		все для билетов продавца, доступен админу и продавцу
3563: 				Без фильтра выводятся только active=1 записи
3564: 				Ответ сервера:
3565: 				{'code':'200','status':'success',
3566: 				"data":{		
3567: 						"cart":[
3568: 							{
3569: 								"u_id":			"идентификатор покупателя для filter=all,trip",
3570: 								"active":		"удалено (0) или нет (1) для filter=all,trip",
3571: 								"prod":			"идентификатор расписания",
3572: 								"prop":			"block места или '' для любого",
3573: 								"count":		"количество мест",
3574: 								"created":		"дата создания"
3575: 								"notice":		"0 или 1, посылать сообщение на почту при появлении билета или нет"
3576: 								"status":		"кастомный статус"
3577: 								"status_sys":	"системный статус"			data.cart_block_statuses_sys
3578: 								"statuses":		[
3579: 													{
3580: 														"t_id":			"идентификатор рейса, для filter=all,trip и продавца только его"
3581: 														"status":		"кастомный статус"
3582: 														"status_sys":	"системный статус"			data.cart_block_statuses_sys
3583: 													}
3584: 												]
3585: 							},	
3586: 							...
3587: 						]
3588: 				},
3589: 				"auth_user":{
3590: 					"u_id":				"идентификатор пользователя",
3591: 					"u_name":			"имя пользователя",
3592: 					"u_family":			"фамилия пользователя",
3593: 					"u_middle":			"отчество пользователя",
3594: 					"u_email":			"емейл пользователя",
3595: 					"u_phone":			"телефон пользователя",
3596: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
3597: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
3598: 					"u_ban":			{
3599: 										"auth":			"число активных банов на авторизацию",
3600: 										"order":		"число активных банов на создание или получения поездки",
3601: 										"blog_topic":	"число активных банов на создание темы в блоге",
3602: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
3603: 										}
```

### `get_fields` — `archive_17012026_1259/taxi/controllers/c_index.php`
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
8103: 					}
8104: 					type_for_get_fields_par_for_trip.onclick = get_fields;
8105: 					role_for_get_fields_par_for_trip.onclick = get_fields;
8106: 					get_fields();
8107: 
8108: 					arr_for_get_fields_par_for_trip.onclick = function(){	
8109: 						var value = {};
8110: 						var max_index = -1;
8111: 						for(var i = 0; i < this.selectedOptions.length; i++){
8112: 							var index_digit = this.selectedOptions[i].value.split(",");
8113: 							var index = index_digit[0];
8114: 							var digit = index_digit[1];
8115: 							if (!(index in value)) {
8116: 								value[index] = {};
8117: 								if (index > max_index) max_index = index;
8118: 							}
8119: 							value[index][digit] = true;
8120: 						}
8121: 						var value_str = "";
8122: 						for(var i = 0; i <= max_index; i++){
8123: 							if (i in value){
8124: 								var digit = sum_keys(value[i]);
8125: 								value_str += digit in digits_chars ? digits_chars[digit] : "0";
8126: 							} else {
8127: 								value_str += "0";
8128: 							}
8129: 						}
8130: 
8131: 						res_for_get_fields_par_for_trip.textContent = value_str;
8132: 					};
8133: 				})();
8134: 			</script>
8135: 
8136: 		https://ibronevik.ru/taxi/api/v1/trip/							GET
8137: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id1,t_id5}			GET
8138: 			Разрешены для админа
8139: 
8140: 		https://ibronevik.ru/taxi/api/v1/cart?prod={t_id}&prop={seat}[&count=1]
8141: 			Добавлена проверка ticket_status при бронировании и продлении.
8142: 
8143: 		https://ibronevik.ru/taxi/api/v1/cart
8144: 			Добавлена проверка ticket_status для busy.
8145: 			Добавлен GET или POST параметр filter.
8146: 			Добавлено:
8147: 				u_id
8148: 
8149: 		https://ibronevik.ru/taxi/api/v1/drive							POST
8150: 			Добавлена проверка ticket_status
8151: 
8152: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/edit/	POST
8153: 			Добавлено редактирование номера и статуса билета.
8154: 			Добавлена отправке оповещений при появлении свободных блоков при смене статуса.
8155: 			
8156: 		https://ibronevik.ru/taxi/api/v1/schedule/ticket				GET
8157: 			Добавлена проверка ticket_status
8158: 	
8159: 		Шаблоны емейлов:
8160: 			Успешная оплата:
8161: 				в {$ticket} добавлен "number":
8162: 					[
8163: 						{
8164: 							"schedule":{
8165: 								"sc_id":"...",
8166: 								"team1":"...",
8167: 								"team2":"...",
8168: 								"stadium":"...",
8169: 								"datetime":"...",
8170: 								"tournament":"...",
8171: 								"str":"одной строкой"
8172: 							},
8173: 							"count": "число билетов sc_id",
8174: 							"trips":[
8175: 								{
8176: 									"t_id":"",
8177: 									"seats":[
8178: 										"block":"...",
8179: 										"row":"...",
8180: 										"seat":"...",
8181: 										"price":"...",
8182: 										"file":"0|1",
8183: 										"number":"..."
8184: 									]
8185: 								}
8186: 							]
8187: 						}
8188: 					]
8189: 
8190: 		https://ibronevik.ru/taxi/api/v1/cart_block
8191: 			Добавлен GET или POST параметр filter.
8192: 			Добавлено:
8193: 				u_id
8194: 				active
8195: 				status
8196: 				status_sys
8197: 				statuses
8198: 
8199: 		https://ibronevik.ru/taxi/api/v1/cart_block/clear
8200: 		https://ibronevik.ru/taxi/api/v1/cart_block?prod={t_id}&prop={seat}[&count={count}][&amp;notice={0|1}]
8201: 			Вместо удаления записей меняется активность
8202: 
8203: 		https://ibronevik.ru/taxi/api/v1/cart_block/status
8204: 			Добавлен метод редактирования статусов корзины оповещений
8205: 		</pre>
8206: 		<pre>
8207: 	Обновление 2 (декабрь 2023)
8208: 		Интут для картинок с лого:
8209: 		<input type="file" id="file_for_team_upload" multiple> <span id="res_for_team_upload_find" style="display:inline-block;vertical-align:top;line-height:0;"></span>
8210: 		<button id="team_upload_create_json">Показать json для редактирования данных</button>
8211: 		<span id="res_for_team_upload_create_json" style="display:inline-block;vertical-align:top;width:95%;word-break:break-all;white-space:normal;"></span>
8212: 		<script>
8213: 			(function(){
8214: 				team_upload_create_json.onclick = function(){
8215: 					var country = country_for_team_clone_find.value;
8216: 					var form_str = {"teams":[]};
8217: 					res_for_team_upload_find.querySelectorAll(':checked[type="checkbox"]').forEach(function(el){
8218: 						var obj = {};
8219: 						obj["en"] = el.value;
8220: 						if (el.nextElementSibling.getAttribute("load")) obj["logo"] = el.nextElementSibling.src;
8221: 						var logo_db = el.nextElementSibling.nextElementSibling && el.nextElementSibling.nextElementSibling.querySelector(':checked');
8222: 						if (logo_db) obj["id"] = logo_db.value;
8223: 						form_str["teams"].push(obj);
8224: 					});
8225: 					res_for_team_upload_create_json.textContent = JSON.stringify(form_str);
8226: 				};
8227: 				file_for_team_upload.onchange = async function(){
8228: 					if (!this.files.length) return true;
8229: 					var f_list = {};
8230: 					for (var i = 0; i < this.files.length; i++){
8231: 						var f = this.files[i];
8232: 						f_list[f.name] = await new Promise(function(resolve){
8233: 							var reader = new FileReader();
8234: 							reader.onload = function(e){resolve(e.target.result);};
8235: 							reader.readAsDataURL(f);
8236: 						});
8237: 					}
8238: 					var coincidences = [];
8239: 					list = Object.keys(f_list);
8240: 					for(var i in list){
8241: 						var str = list[i].replace(/\.[^.]+$/,"").replace(/_/g," ").trim();
8242: 						if (str){
8243: 							var label = str;
8244: 							coincidences[label] = [];
8245: 							str = str.toLowerCase().split(/[\s'-]+/);
8246: 							var reg_add = [];
8247: 							if (str.length > 1){
8248: 								var reg = [];
8249: 								for(var j in str){
8250: 									var reg_inessential = new RegExp('(^|\\s)(Al|BSC|CD|CF|City|Club|D\.?|FC|FSV|KC|LA|NY|RB|RC|RCD|SC|SD|SJ|TSG|UD|United|VfB|VfL)(\\s|$)','i');
8251: 									if (!reg_inessential.test(str[j])){
8252: 										reg.push(str[j].replace(/[^a-z0-9]+/,function($0){
8253: 											return ".{0," + $0.length + "}";
8254: 										}));
8255: 										reg_add_el = str[j].split(/[^a-z0-9]+/);
8256: 										for(var index in reg_add_el){
8257: 											if (reg_add_el[index].length >= 5){
8258: 												reg_add.push("((^|[^a-z0-9])" + reg_add_el[index] + "|" + reg_add_el[index] + "([^a-z0-9]|$))");
8259: 											}
8260: 										}
8261: 									}
8262: 								}
8263: 								if (reg.length){
8264: 									reg = new RegExp('(^|\\s|,|\\.|\\!|\'|-)(' + reg.join("|") + ')(\\s|,|\\.|\\!|\'|-|$)','i');
8265: 								} else {
8266: 									coincidences[label].push("warning!");
8267: 									continue;
8268: 								}
8269: 							} else {
8270: 								var part = str[0];
8271: 								var reg = new RegExp('(^|\\s|,|\\.|\\!|\'|-)' +  part.replace(/[^a-z0-9]+/,function($0){
8272: 									return ".{0," + $0.length + "}";
8273: 								}) + '(\\s|,|\\.|\\!|\'|-|$)','i');
8274: 								reg_add_el = part.split(/[^a-z0-9]+/);
8275: 								for(var index in reg_add_el){
8276: 									if (reg_add_el[index].length >= 5){
8277: 										reg_add.push("((^|[^a-z0-9])" + reg_add_el[index] + "|" + reg_add_el[index] + "([^a-z0-9]|$))");
8278: 									}
8279: 								}
8280: 							}
8281: 							coincidences[label].push('<img onload="this.setAttribute(\'load\',1);" style="vertical-align:top;" i="' + i + '" src="' + f_list[list[i]] + '">');
8282: 							if (reg_add.length){
8283: 								reg_add = new RegExp(reg_add.join("|"),'i');
8284: 								for(var id in data_teams){				
8285: 									if (reg.test(data_teams[id].en)){
8286: 										coincidences[label].push('<input val="0" onclick="if(this.getAttribute(\'val\')==1){this.checked=false;this.setAttribute(\'val\',0);}else{this.setAttribute(\'val\',1);}" name="file_for_team_upload' + i + '" type="radio" value="' + id + '" style="display:inline-block;vertical-align:top;margin:0 2px 0 0;">'+data_teams[id].en.replace(/[&<>]g/,function($0){
8287: 											var template = {"&":"&amp;","<":"&lt;",">":"&gt;"};
8288: 											return template[$0];
8289: 										}) + (data_teams[id].logo ? '<img style="vertical-align:top;" src="' + data_teams[id].logo + '">' : ""));
8290: 									} else {
8291: 										if (reg_add.test(data_teams[id].en)){
8292: 											coincidences[label].push('<input val="0" onclick="if(this.getAttribute(\'val\')==1){this.checked=false;this.setAttribute(\'val\',0);}else{this.setAttribute(\'val\',1);}" name="file_for_team_upload' + i + '" type="radio" value="' + id + '" style="display:inline-block;vertical-align:top;margin:0 2px 0 0;">'+data_teams[id].en.replace(/[&<>]g/,function($0){
8293: 												var template = {"&":"&amp;","<":"&lt;",">":"&gt;"};
8294: 												return template[$0];
8295: 											}) + (data_teams[id].logo ? '<img style="vertical-align:top;" src="' + data_teams[id].logo + '">' : ""));
8296: 										}
8297: 									}
8298: 								}
8299: 							} else {
8300: 								for(var id in data_teams){	
8301: 									if (reg.test(data_teams[id].en)){
8302: 										coincidences[label].push('<input val="0" onclick="if(this.getAttribute(\'val\')==1){this.checked=false;this.setAttribute(\'val\',0);}else{this.setAttribute(\'val\',1);}" name="file_for_team_upload' + i + '" type="radio" value="' + id + '" style="display:inline-block;vertical-align:top;margin:0 2px 0 0;">'+data_teams[id].en.replace(/[&<>]g/,function($0){
8303: 											var template = {"&":"&amp;","<":"&lt;",">":"&gt;"};
8304: 											return template[$0];
8305: 										}) + (data_teams[id].logo ? '<img style="vertical-align:top;" src="' + data_teams[id].logo + '">' : ""));
8306: 									}
8307: 								}
8308: 							}
8309: 						}
8310: 					}
8311: 					var html = [];
8312: 					for(var label in coincidences){
8313: 						var img_html = coincidences[label][0];
8314: 						coincidences[label] = coincidences[label].slice(1);
8315: 						if (coincidences[label].length){
8316: 							html.push('<span style="line-height:normal;display:block;margin-bottom:5px;border:1px solid;padding:2px;border-radius:5px;"><input type="checkbox" value="' + label.replace(/[&"]/g,function($0){var template = {"&":"&amp;","\"":"&quot;"};return template[$0];}) + '" style="display:inline-block;vertical-align:top;margin:0 2px 0 0;">' + label.replace(/[&<>]g/,function($0){var template = {"&":"&amp;","<":"&lt;",">":"&gt;"};return template[$0];}) + img_html + ':\t\t<span style="display:inline-block;vertical-align:top;border-left:1px solid;"><span style="display:block;">' + coincidences[label].join('</span><hr><span style="display:block;">')+ '</span></span></span>');
8317: 						} else {
8318: 							html.push('<span style="line-height:normal;display:block;margin-bottom:5px;border:1px solid;padding:2px;border-radius:5px;"><input type="checkbox" value="' + label.replace(/[&"]/g,function($0){var template = {"&":"&amp;","\"":"&quot;"};return template[$0];}) + '" style="display:inline-block;vertical-align:top;margin:0 2px 0 0;">' + label.replace(/[&<>]g/,function($0){var template = {"&":"&amp;","<":"&lt;",">":"&gt;"};return template[$0];}) + img_html + '</span>');
8319: 						}
8320: 					}
8321: 					res_for_team_upload_find.innerHTML = html.join("\n");
8322: 
8323: 				};
8324: 			})();
```

### `queryString` — `archive_17012026_1259/taxi/models/api.php`
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
18048: 
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
```

## 4. Критерий классификации

```text
explicit ALLOW / REJECT
        ↓
AUTHORIZATION

SQL predicate / WHERE / JOIN / data filtering
        ↓
QUERY_SCOPE
```

Если consumer только передаёт `query_roles` в другой query builder, это ещё не конечное Evidence. Нужно идти по delegation chain до места, где создаётся predicate или принимается access decision.

## 5. Текущий вывод

В доступном source corpus `query_roles` участвует в query-related processing. Для каждого consumer необходимо различать прямое использование и delegation.

Пока не фиксируем:

```text
Role → CAN_EXECUTE → Query
query_roles → Authorization Policy
```

до нахождения явного ALLOW/REJECT.

## 6. Research Loop result

RP-12 уточняет UNKNOWN до следующего уровня:

```text
query_roles
   ↓
consumer
   ↓
delegation / query builder
   ↓
SQL predicate OR explicit access decision
```

## 7. Gap Report

```text
G-12-01  exact SQL predicate/result effect     OPEN
G-12-02  explicit permission rejection         OPEN
G-12-03  Role ID → business role              OPEN
G-12-04  complete Role × Operation matrix     OPEN
```

## 8. MCR

`MCR = NO CHANGE`.

## 9. Следующий шаг

Если consumer делегирует query construction, следующий pass должен исследовать именно target function этого delegation. Если SQL predicate уже виден, закрыть G-12-01 на уровне конкретного predicate/result. Не расширять область исследования.