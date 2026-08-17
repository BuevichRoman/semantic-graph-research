# Backend Semantic Graph — Research Pass 06
# Role Permission Matrix v0.1

**Статус:** PARTIALLY ANSWERED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-05 Role Resolution Trace v0.2  
**Цель:** собрать доказанную матрицу `Role × Protected Operation` только из прямых role checks production code.

## 1. Правило прохода

RP-06 не выводит разрешения из названий ролей, endpoint'ов или бизнес-смысла. В матрицу попадают только операции, для которых в коде найден явный role check или явная ветка `wrong user role`. Если конкретная роль не встречается в проверке, это не считается автоматически `ALLOW`.

## 2. Найденные прямые проверки

### archive_17012026_1259/taxi/models/api.php:25
```text
24: 					{
25: 						return $this->showError('404', 'error', 'wrong user role');
26: 					}
```

### archive_17012026_1259/taxi/models/api.php:30
```text
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
```

### archive_17012026_1259/taxi/models/api.php:37
```text
36: 			{
37: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
38: 				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
```

### archive_17012026_1259/taxi/models/api.php:661
```text
660: 			{	
661: 				if ($this->id_role != 4)
662: 				{
```

### archive_17012026_1259/taxi/models/api.php:685
```text
684: 					$prop_visibility = 8;
685: 					if ($this->id_role != 4) 
686: 					{
```

### archive_17012026_1259/taxi/models/api.php:1051
```text
1050: 					default:
1051: 						return $this->showError('404', 'error', 'wrong user role');
1052: 				}
```

### archive_17012026_1259/taxi/models/api.php:1056
```text
1055: 			{
1056: 				if ($this->id_role != 4) 
1057: 				{
```

### archive_17012026_1259/taxi/models/api.php:1379
```text
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
```

### archive_17012026_1259/taxi/models/api.php:1487
```text
1486: 						{
1487: 							return $this->showError('404', 'error', 'wrong user role');
1488: 						}
```

### archive_17012026_1259/taxi/models/api.php:1990
```text
1989: 					$d['u_id'] = explode(',',$d['u_id']);
1990: 					if ($this->id_role != 4 && !in_array($_SESSION[UID],$d['u_id']))
1991: 					{
```

### archive_17012026_1259/taxi/models/api.php:1997
```text
1996: 				{
1997: 					if ($this->id_role != 4) unset($d['details']);
1998: 				}
```

### archive_17012026_1259/taxi/models/api.php:2145
```text
2144: 				{
2145: 					if ($this->id_role == 2)
2146: 					{
```

### archive_17012026_1259/taxi/models/api.php:2164
```text
2163: 					{
2164: 						return $this->showError('404', 'error', 'wrong user role');
2165: 					}
```

### archive_17012026_1259/taxi/models/api.php:2169
```text
2168: 				{
2169: 					if ($this->id_role != 4) 
2170: 					{
```

### archive_17012026_1259/taxi/models/api.php:2188
```text
2187: 					if (empty($d['id_user'])) return $this->showError('404', 'error', 'user not found');
2188: 					if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong role of user');
2189: 					
```

### archive_17012026_1259/taxi/models/api.php:2200
```text
2199: 			{			
2200: 				if (!empty($id_user) && $this->id_role != 4) 
2201: 				{
```

### archive_17012026_1259/taxi/models/api.php:2225
```text
2224: 				{
2225: 					if ($this->id_role == 4 || (in_array($_SESSION[UID], explode(',',$d['u_id'])) 
2226: 						&& (empty($_SESSION['id_verification_status']) || $_SESSION['id_verification_status'] == 1)))
```

### archive_17012026_1259/taxi/models/api.php:2255
```text
2254: 					{
2255: 						return $this->showError('404', 'error', 'not enough rights or wrong user role');
2256: 					}
```

### archive_17012026_1259/taxi/models/api.php:2849
```text
2848: 			}
2849: 			if ($this->id_role != 1 && $this->id_role != 5 && $this->id_role != 4)
2850: 			{
```

### archive_17012026_1259/taxi/models/api.php:2851
```text
2850: 			{
2851: 				return $this->showError('404', 'error', 'wrong user role');
2852: 			}
```

### archive_17012026_1259/taxi/models/api.php:3471
```text
3470: 			{
3471: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
3472: 
```

### archive_17012026_1259/taxi/models/api.php:3990
```text
3989: 			{
3990: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'forbidden payment way');
3991: 			}
```

### archive_17012026_1259/taxi/models/api.php:4481
```text
4480: 
4481: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
4482: 			{
```

### archive_17012026_1259/taxi/models/api.php:4483
```text
4482: 			{
4483: 				return $this->showError('404', 'error', 'wrong user role');
4484: 			}
```

### archive_17012026_1259/taxi/models/api.php:4545
```text
4544: 
4545: 			if ($this->id_role == 1 || $this->id_role == 5)
4546: 			{
```

### archive_17012026_1259/taxi/models/api.php:4775
```text
4774: 
4775: 						if ($this->id_role == 1 || $this->id_role == 5)
4776: 						{
```

### archive_17012026_1259/taxi/models/api.php:4818
```text
4817: 
4818: 				if ($this->id_role == 1 || $this->id_role == 5)
4819: 				{
```

### archive_17012026_1259/taxi/models/api.php:4852
```text
4851: 				$d['b_options'] = json_decode($d['b_options'],true);
4852: 				if (($this->id_role != 1 && $this->id_role != 5) && is_array($d['b_options']))
4853: 				{
```

### archive_17012026_1259/taxi/models/api.php:4948
```text
4947: 			}
4948: 			if ($this->id_role != 2 && $this->id_role != 4)
4949: 			{
```

### archive_17012026_1259/taxi/models/api.php:4950
```text
4949: 			{
4950: 				return $this->showError('404', 'error', 'wrong user role');
4951: 			}
```

### archive_17012026_1259/taxi/models/api.php:5003
```text
5002: 
5003: 			if ($this->id_role == 4)
5004: 			{
```

### archive_17012026_1259/taxi/models/api.php:5020
```text
5019: 			} 
5020: 			elseif ($this->id_role == 2)
5021: 			{
```

### archive_17012026_1259/taxi/models/api.php:5067
```text
5066: 			$union = false;
5067: 			if (isset($filter) && $this->id_role == 2)
5068: 			{
```

### archive_17012026_1259/taxi/models/api.php:5356
```text
5355: 			}
5356: 			elseif (isset($filter) && $this->id_role == 2 && $check_license === true)
5357: 			{
```

### archive_17012026_1259/taxi/models/api.php:5512
```text
5511: 
5512: 						if ($this->id_role == 4)
5513: 						{
```

### archive_17012026_1259/taxi/models/api.php:5524
```text
5523: 
5524: 				if ($this->id_role == 4)
5525: 				{
```

### archive_17012026_1259/taxi/models/api.php:5555
```text
5554: 				$d['b_options'] = json_decode($d['b_options'],true);
5555: 				if ($this->id_role == 2 && is_array($d['b_options']))
5556: 				{
```

### archive_17012026_1259/taxi/models/api.php:5608
```text
5607: 			}
5608: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
5609: 			{
```

### archive_17012026_1259/taxi/models/api.php:5610
```text
5609: 			{
5610: 				return $this->showError('404', 'error', 'wrong user role');
5611: 			}
```

### archive_17012026_1259/taxi/models/api.php:5672
```text
5671: 		
5672: 			if ($this->id_role == 1 || $this->id_role == 5)
5673: 			{
```

### archive_17012026_1259/taxi/models/api.php:5911
```text
5910: 						$d['drivers'][$key] = array();
5911: 						if ($this->id_role == 1 || $this->id_role == 5)
5912: 						{
```

### archive_17012026_1259/taxi/models/api.php:6011
```text
6010: 
6011: 						if ($this->id_role == 1 || $this->id_role == 5)
6012: 						{
```

### archive_17012026_1259/taxi/models/api.php:6053
```text
6052: 				}
6053: 				if ($this->id_role == 1 || $this->id_role == 5)
6054: 				{
```

### archive_17012026_1259/taxi/models/api.php:6084
```text
6083: 				$d['b_options'] = json_decode($d['b_options'],true);
6084: 				if (($this->id_role != 1 && $this->id_role != 5) && is_array($d['b_options']))
6085: 				{
```

### archive_17012026_1259/taxi/models/api.php:6094
```text
6093: 				add_time_zone($d['b_created']);
6094: 				if ($this->id_role == 1 || $this->id_role == 5)
6095: 				{
```

### archive_17012026_1259/taxi/models/api.php:6186
```text
6185: 
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
```

### archive_17012026_1259/taxi/models/api.php:6188
```text
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
6189: 			}
```

### archive_17012026_1259/taxi/models/api.php:6191
```text
6190: 			
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
```

### archive_17012026_1259/taxi/models/api.php:6804
```text
6803: 			}
6804: 			if ($this->id_role != 2)
6805: 			{
```

### archive_17012026_1259/taxi/models/api.php:6806
```text
6805: 			{
6806: 				return $this->showError('404', 'error', 'wrong user role');
6807: 			}
```

### archive_17012026_1259/taxi/models/api.php:6901
```text
6900: 			}
6901: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6902: 			{
```

### archive_17012026_1259/taxi/models/api.php:6903
```text
6902: 			{
6903: 				return $this->showError('404', 'error', 'wrong user role');
6904: 			}
```

### archive_17012026_1259/taxi/models/api.php:6906
```text
6905: 
6906: 			if ($this->id_role == 1 || $this->id_role == 5)
6907: 			{
```

### archive_17012026_1259/taxi/models/api.php:6949
```text
6948: 			}
6949: 			elseif ($this->id_role == 2)
6950: 			{		
```

### archive_17012026_1259/taxi/models/api.php:7055
```text
7054: 			}
7055: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5 && $this->id_role != 4)
7056: 			{
```

### archive_17012026_1259/taxi/models/api.php:7057
```text
7056: 			{
7057: 				return $this->showError('404', 'error', 'wrong user role');
7058: 			}
```

### archive_17012026_1259/taxi/models/api.php:7082
```text
7081: 	
7082: 			if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
7083: 			{
```

### archive_17012026_1259/taxi/models/api.php:7106
```text
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
7107: 				{
```

### archive_17012026_1259/taxi/models/api.php:7401
```text
7400: 			}
7401: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
7402: 			{
```

### archive_17012026_1259/taxi/models/api.php:7403
```text
7402: 			{
7403: 				return $this->showError('404', 'error', 'wrong user role');
7404: 			}
```

### archive_17012026_1259/taxi/models/api.php:7406
```text
7405: 
7406: 			if ($this->id_role == 1 || $this->id_role == 5)
7407: 			{
```

### archive_17012026_1259/taxi/models/api.php:7531
```text
7530: 			}
7531: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
7532: 			{
```

### archive_17012026_1259/taxi/models/api.php:7533
```text
7532: 			{
7533: 				return $this->showError('404', 'error', 'wrong user role');
7534: 			}			
```

### archive_17012026_1259/taxi/models/api.php:7594
```text
7593: 
7594: 			if ($this->id_role == 1 || $this->id_role == 4 || $this->id_role == 5)
7595: 			{
```

### archive_17012026_1259/taxi/models/api.php:7618
```text
7617: 			}
7618: 			elseif ($this->id_role == 2)
7619: 			{
```

### archive_17012026_1259/taxi/models/api.php:7639
```text
7638: 
7639: 			if ($this->id_role == 2 || $this->id_role == 4)
7640: 			{
```

### archive_17012026_1259/taxi/models/api.php:7649
```text
7648: 
7649: 			if ($this->id_role == 2)
7650: 			{
```

### archive_17012026_1259/taxi/models/api.php:7678
```text
7677: 			}
7678: 			elseif ($this->id_role == 1 || $this->id_role == 5)
7679: 			{
```

### archive_17012026_1259/taxi/models/api.php:7875
```text
7874: 						$d['drivers'][$key] = array();
7875: 						if ($this->id_role == 1 || $this->id_role == 5)
7876: 						{
```

### archive_17012026_1259/taxi/models/api.php:7928
```text
7927: 								}
7928: 								if ($this->id_role != 4 && $_SESSION[UID] != $d['drivers'][$key]['u_id'])
7929: 								{
```

### archive_17012026_1259/taxi/models/api.php:7974
```text
7973: 						
7974: 						if ($this->id_role == 4)
7975: 						{
```

### archive_17012026_1259/taxi/models/api.php:7978
```text
7977: 						}
7978: 						elseif ($this->id_role == 1 || $this->id_role == 5)
7979: 						{
```

### archive_17012026_1259/taxi/models/api.php:8020
```text
8019: 				}
8020: 				if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
8021: 				{
```

### archive_17012026_1259/taxi/models/api.php:8055
```text
8054: 				$d['b_options'] = json_decode($d['b_options'],true);
8055: 				if ($this->id_role == 2 && is_array($d['b_options']))
8056: 				{
```

### archive_17012026_1259/taxi/models/api.php:8069
```text
8068: 				add_time_zone($d['b_created']);
8069: 				if ($this->id_role == 1 || $this->id_role == 4 || $this->id_role == 5)
8070: 				{
```

### archive_17012026_1259/taxi/models/api.php:8186
```text
8185: 			}
8186: 			if ($this->id_role != 1 && $this->id_role != 5)
8187: 			{
```

### archive_17012026_1259/taxi/models/api.php:8188
```text
8187: 			{
8188: 				return $this->showError('404', 'error', 'wrong user role');
8189: 			}
```

### archive_17012026_1259/taxi/models/api.php:8253
```text
8252: 			}
8253: 			if ($this->id_role != 2)
8254: 			{
```

### archive_17012026_1259/taxi/models/api.php:8255
```text
8254: 			{
8255: 				return $this->showError('404', 'error', 'wrong user role');
8256: 			}
```

### archive_17012026_1259/taxi/models/api.php:8744
```text
8743: 			}
8744: 			if ($this->id_role != 1 && $this->id_role != 5)
8745: 			{
```

### archive_17012026_1259/taxi/models/api.php:8746
```text
8745: 			{
8746: 				return $this->showError('404', 'error', 'wrong user role');
8747: 			}
```

### archive_17012026_1259/taxi/models/api.php:8972
```text
8971: 			}
8972: 			if ($this->id_role != 2)
8973: 			{
```

### archive_17012026_1259/taxi/models/api.php:8974
```text
8973: 			{
8974: 				return $this->showError('404', 'error', 'wrong user role');
8975: 			}
```

### archive_17012026_1259/taxi/models/api.php:9256
```text
9255: 
9256: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
9257: 			{
```

### archive_17012026_1259/taxi/models/api.php:9258
```text
9257: 			{
9258: 				return $this->showError('404', 'error', 'wrong user role');
9259: 			}
```

### archive_17012026_1259/taxi/models/api.php:9312
```text
9311: 
9312: 			if ($this->id_role == 1 || $this->id_role == 5)
9313: 			{
```

### archive_17012026_1259/taxi/models/api.php:10662
```text
10661: 				$id_user = $_SESSION[UID];
10662: 				if ($this->id_role != 4) $sql_add = "";
10663: 			}
```

### archive_17012026_1259/taxi/models/api.php:10666
```text
10665: 			{	
10666: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10667: 			}
```

### archive_17012026_1259/taxi/models/api.php:10804
```text
10803: 			{	
10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10805: 				$s = "SELECT
```

### archive_17012026_1259/taxi/models/api.php:10891
```text
10890: 			{	
10891: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10892: 				$s = "SELECT
```

### archive_17012026_1259/taxi/models/api.php:10935
```text
10934: 			{	
10935: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10936: 
```

### archive_17012026_1259/taxi/models/api.php:11086
```text
11085: 				$id_user = $_SESSION[UID];
11086: 				if ($this->id_role != 4) $sql_add = "";
11087: 			}
```

### archive_17012026_1259/taxi/models/api.php:11090
```text
11089: 			{	
11090: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11091: 			}
```

### archive_17012026_1259/taxi/models/api.php:11206
```text
11205: 			}
11206: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
11207: 			{
```

### archive_17012026_1259/taxi/models/api.php:11208
```text
11207: 			{
11208: 				return $this->showError('404', 'error', 'wrong user role');
11209: 			}
```

### archive_17012026_1259/taxi/models/api.php:11211
```text
11210: 
11211: 			if ($this->id_role == 1 || $this->id_role == 5)
11212: 			{
```

### archive_17012026_1259/taxi/models/api.php:11373
```text
11372: 			{
11373: 				if ($this->id_role != 2)
11374: 				{
```

### archive_17012026_1259/taxi/models/api.php:11375
```text
11374: 				{
11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
11376: 				}
```

### archive_17012026_1259/taxi/models/api.php:11392
```text
11391: 			{
11392: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11393: 				$s = "SELECT 
```

### archive_17012026_1259/taxi/models/api.php:12137
```text
12136: 			{
12137: 				if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
12138: 				{
```

### archive_17012026_1259/taxi/models/api.php:12139
```text
12138: 				{
12139: 					if ($type !== NULL) return $this->showError('404', 'error', 'wrong user role');
12140: 				}
```

### archive_17012026_1259/taxi/models/api.php:12229
```text
12228: 			}
12229: 			if ($this->id_role == 2 || $this->id_role == 4)
12230: 			{
```

### archive_17012026_1259/taxi/models/api.php:12242
```text
12241: 				$sql_left_join = "LEFT JOIN `order_trip` USING (`id_trip`)";
12242: 				if ($this->id_role == 2) $sql_where .= " AND `trip`.`driver` = '" . $_SESSION[UID] . "'";
12243: 			}
```

### archive_17012026_1259/taxi/models/api.php:12244
```text
12243: 			}
12244: 			elseif ($this->id_role == 1 || $this->id_role == 5)
12245: 			{
```

### archive_17012026_1259/taxi/models/api.php:12250
```text
12249: 			{
12250: 				if ($this->id_role == 1 || $this->id_role == 5)
12251: 				{
```

### archive_17012026_1259/taxi/models/api.php:12281
```text
12280: 			{
12281: 				if ($this->id_role == 1 || $this->id_role == 5)
12282: 				{
```

### archive_17012026_1259/taxi/models/api.php:12302
```text
12301: 			{
12302: 				if ($this->id_role == 1 || $this->id_role == 5)
12303: 				{
```

### archive_17012026_1259/taxi/models/api.php:12592
```text
12591: 						);						
12592: 						if ($this->id_role != 2 && $this->id_role != 4)
12593: 						{
```

### archive_17012026_1259/taxi/models/api.php:12886
```text
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
```

### archive_17012026_1259/taxi/models/api.php:12888
```text
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
12889: 			}
```

### archive_17012026_1259/taxi/models/api.php:12982
```text
12981: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
12982: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
12983: 			{
```

### archive_17012026_1259/taxi/models/api.php:13572
```text
13571: 			}
13572: 			if ($this->id_role != 1 && $this->id_role != 5)
13573: 			{
```

### archive_17012026_1259/taxi/models/api.php:13574
```text
13573: 			{
13574: 				return $this->showError('404', 'error', 'wrong user role');
13575: 			}
```

### archive_17012026_1259/taxi/models/api.php:13730
```text
13729: 				{
13730: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
13731: 
```

### archive_17012026_1259/taxi/models/api.php:13751
```text
13750: 				$id_dropbox_link = $file['dl_id'];
13751: 				if ($this->id_role == 4)
13752: 				{
```

### archive_17012026_1259/taxi/models/api.php:14010
```text
14009: 			}
14010: 			elseif ($this->id_role == 4)
14011: 			{
```

### archive_17012026_1259/taxi/models/api.php:14346
```text
14345: 
14346: 			if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
14347: 			{
```

### archive_17012026_1259/taxi/models/api.php:16850
```text
16849: 			{
16850: 				if ($this->id_role != 4)
16851: 				{
```

### archive_17012026_1259/taxi/models/api.php:17403
```text
17402: 			}
17403: 			if ($this->id_role != 4)
17404: 			{
```

### archive_17012026_1259/taxi/models/api.php:17449
```text
17448: 			{
17449: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
17450: 				$sql_where = "1 = 1";
```

### archive_17012026_1259/taxi/models/api.php:17454
```text
17453: 			{
17454: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter');
17455: 				$sql_where = "`trip`.`driver` = '" . $_SESSION[UID] . "'";
```

### archive_17012026_1259/taxi/models/api.php:17892
```text
17891: 			{
17892: 				if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
17893: 				{
```

### archive_17012026_1259/taxi/models/api.php:17911
```text
17910: 				{
17911: 					if ($this->id_role != 4)  return $this->showError('404', 'error', 'not enough rights');
17912: 					if ($hash !=  md5('checking' . md5(API_KEY))) return $this->showError('404', 'error', 'wrong hash');
```

### archive_17012026_1259/taxi/models/api.php:17936
```text
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
```

### archive_17012026_1259/taxi/models/api.php:18136
```text
18135: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18136: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18137: 			{
```

### archive_17012026_1259/taxi/models/api.php:18260
```text
18259: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18260: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18261: 				{
```

### archive_17012026_1259/taxi/models/api.php:18368
```text
18367: 					if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
18368: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18369: 					{
```

### archive_17012026_1259/taxi/models/api.php:18377
```text
18376: 					if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
18377: 					if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18378: 					{
```

### archive_17012026_1259/taxi/models/api.php:18624
```text
18623: 			{
18624: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter all');
18625: 				$sql_field = "`cart_block`.`id_user` as u_id,
```

### archive_17012026_1259/taxi/models/api.php:18627
```text
18626: 				`cart_block`.`active`,";
18627: 				if ($this->id_role != 4)
18628: 				{
```

### archive_17012026_1259/taxi/models/api.php:18641
```text
18640: 			{
18641: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter trip');
18642: 				$sql_field = "`cart_block`.`id_user` as u_id,
```

### archive_17012026_1259/taxi/models/api.php:18644
```text
18643: 				`cart_block`.`active`,";
18644: 				if ($this->id_role != 4)
18645: 				{
```

### archive_17012026_1259/taxi/models/api.php:18958
```text
18957: 
18958: 			if (empty($_SESSION[UID]) || $this->id_role != 2) {
18959: 				if (empty($sc_id)) return $this->showError('404', 'error', 'empty sc_id');
```

### archive_17012026_1259/taxi/models/api.php:19126
```text
19125: 				if (empty($d['id_seat'])) return $this->showError('404', 'error', "seat not found for trip");
19126: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller');
19127: 				if ($ignore_order === false && empty($d['client'])) return $this->showError('404', 'error', 'buyer not found');				
```

### archive_17012026_1259/taxi/models/api.php:19138
```text
19137: 				if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found for trip");
19138: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not seller of trip');
19139: 				if (!array($t_options['seats_sold'][$seat]) || count($t_options['seats_sold'][$seat]) < 2) 
```

### archive_17012026_1259/taxi/models/api.php:19814
```text
19813: 			$is_usher = false;
19814: 			if ($this->id_role != 4 && ($d['driver'] != $_SESSION[UID] || $this->id_role == 6)) 
19815: 			{
```

### archive_17012026_1259/taxi/models/api.php:19816
```text
19815: 			{
19816: 				if ($this->id_role == 6)
19817: 				{
```

### archive_17012026_1259/taxi/models/api.php:19862
```text
19861: 				}
19862: 				elseif ($this->id_role != 4)
19863: 				{
```

### archive_17012026_1259/taxi/models/api.php:20153
```text
20152: 
20153: 			if (!empty($sql_templates[$template]['only_admin']) && $this->id_role != 4)
20154: 			{
```

### archive_17012026_1259/taxi/models/api.php:20196
```text
20195: 									'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
20196: 									'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
20197: 									'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
```

### archive_17012026_1259/taxi/models/api.php:20284
```text
20283: 
20284: 			if (!empty($script_template['only_admin']) && $this->id_role != 4)
20285: 			{
```

### archive_17012026_1259/taxi/models/api.php:20306
```text
20305: 			} else {
20306: 				if ($this->id_role != 4)
20307: 				{
```

### archive_17012026_1259/taxi/models/api.php:20347
```text
20346: 
20347: 			if ($this->id_role != 4)
20348: 			{
```

### archive_17012026_1259/taxi/models/api.php:20349
```text
20348: 			{
20349: 				if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong user role');
20350: 				if ($_SESSION['id_verification_status'] != 2)
```

### archive_17012026_1259/taxi/models/api.php:20413
```text
20412: 				{
20413: 					if ($this->id_role != 4) {$out['warning'][]  = "$i update wrong user role"; continue;}
20414: 	
```

### archive_17012026_1259/taxi/models/api.php:20463
```text
20462: 							$driver = $d['driver'];
20463: 							if ($driver != $_SESSION[UID] && $this->id_role != 4)
20464: 							{
```

### archive_17012026_1259/taxi/models/api.php:20523
```text
20522: 				$id_user = $_SESSION[UID];
20523: 				if ($this->id_role != 4) $sql_add = "";
20524: 			}
```

### archive_17012026_1259/taxi/models/api.php:20527
```text
20526: 			{	
20527: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
20528: 			}
```

### archive_17012026_1259/taxi/models/api.php:20674
```text
20673: 				case '1':
20674: 					if ($this->id_role != 4)
20675: 					{
```

### archive_17012026_1259/taxi/models/api.php:20866
```text
20865: 						`contact_items`.`fromname`";
20866: 			if ($this->id_role != 4) $sql_private_o_type_other = ",NULL as 'number',
20867: 						NULL as 'key1',
```

### archive_17012026_1259/taxi/models/api.php:20880
```text
20879: 			{
20880: 				if ($this->id_role != 4) $id_user = $_SESSION[UID];
20881: 			}
```

### archive_17012026_1259/taxi/models/api.php:20884
```text
20883: 			{
20884: 				if ($this->id_role != 4 && $id_user != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
20885: 			}
```

### archive_17012026_1259/taxi/models/api.php:21264
```text
21263: 				{
21264: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
21265: 				}
```

### archive_17012026_1259/taxi/models/api.php:21268
```text
21267: 				{
21268: 					if ($filtered_data['owner'] != $_SESSION[UID] && $this->id_role != 4) return $this->showError('404', 'error', 'not enough rights for action');
21269: 				}
```

### archive_17012026_1259/taxi/models/api.php:21446
```text
21445: 			if (empty($d['id_contact_item'])) return $this->showError('404', 'error', 'contact not found');
21446: 			if ($this->id_role != 4 && ($d['id_owner_type'] != 1 || $d['owner'] != $_SESSION[UID])) return $this->showError('404', 'error', 'not enough rights');
21447: 
```

### archive_17012026_1259/taxi/models/api.php:21448
```text
21447: 
21448: 			if ($this->id_role != 4)
21449: 			{
```

### archive_17012026_1259/taxi/models/api.php:21838
```text
21837: 					$from_o_type = $contacts[$co_id]['id_owner_type'];
21838: 					if ($this->id_role != 4 && $from_o_type != 1) return $this->showError('404', 'error', 'code: not enough rights');
21839: 					$owner_types_filtered = array($from_o_type => $owner_types[$from_o_type]);
```

### archive_17012026_1259/taxi/models/api.php:22616
```text
22615: 
22616: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22617: 
```

### archive_17012026_1259/taxi/models/api.php:22901
```text
22900: 
22901: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22902: 
```

### archive_17012026_1259/taxi/models/api.php:22932
```text
22931: 
22932: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
22933: 
```

### archive_17012026_1259/taxi/models/api.php:23051
```text
23050: 
23051: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23052: 
```

### archive_17012026_1259/taxi/models/api.php:23184
```text
23183: 
23184: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23185: 
```

### archive_17012026_1259/taxi/models/api.php:23239
```text
23238: 
23239: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
23240: 
```

### archive_17012026_1259/taxi/models/api.php:23390
```text
23389: 			if (!empty($d['deleted'])) return $this->showError('404', 'error', 'dropbox file already deleted');
23390: 			if ($this->id_role != 4 && $d['user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
23391: 			$json = json_decode($d['json'],true);
```

### archive_17012026_1259/taxi/models/api.php:23473
```text
23472: 			}
23473: 			elseif ($this->id_role == 4)
23474: 			{
```

### archive_17012026_1259/taxi/models/api.php:23617
```text
23616: 			$sql_where_add = " AND (`ticket`.`id_order` IS NULL AND `cart`.`id_user` IS NULL AND `ticket`.`status` not in (2,3))";
23617: 			if ($taken == true && ($this->id_role == 1 || $this->id_role == 2 || $this->id_role == 4))
23618: 			{
```

### archive_17012026_1259/taxi/models/api.php:23619
```text
23618: 			{
23619: 				if ($this->id_role == 4)
23620: 				{
```

### archive_17012026_1259/taxi/models/api.php:23629
```text
23628: 				}
23629: 				elseif ($this->id_role == 1)
23630: 				{
```

### archive_17012026_1259/taxi/models/api.php:23639
```text
23638: 				}
23639: 				elseif ($this->id_role == 2)
23640: 				{
```

### archive_17012026_1259/taxi/models/api.php:23650
```text
23649: 			{
23650: 				if ($this->id_role == 4)
23651: 				{
```

### archive_17012026_1259/taxi/models/api.php:23660
```text
23659: 				}
23660: 				elseif ($this->id_role == 1)
23661: 				{
```

### archive_17012026_1259/taxi/models/api.php:23670
```text
23669: 				}
23670: 				elseif ($this->id_role == 2)
23671: 				{
```

### archive_17012026_1259/taxi/models/api.php:24140
```text
24139: 			{		
24140: 				if ($this->id_role == 1 && !empty($ticket['b_id']) && empty($out['booking'][$ticket['b_id']]['b_payment_datetime']))
24141: 				{
```

### archive_17012026_1259/taxi/models/api.php:24234
```text
24233: 			}
24234: 			if ($this->id_role != 4 && $this->id_role != 6) 
24235: 			{
```

### archive_17012026_1259/taxi/models/api.php:24245
```text
24244: 
24245: 			if ($this->id_role == 6)
24246: 			{
```

### archive_17012026_1259/taxi/models/api.php:24456
```text
24455: 
24456: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24457: 			
```

### archive_17012026_1259/taxi/models/api.php:24772
```text
24771: 			{
24772: 				if ($this->id_role != 4) 
24773: 				{
```

### archive_17012026_1259/taxi/models/api.php:24910
```text
24909: 				{
24910: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
24911: 				}
```

### archive_17012026_1259/taxi/models/api.php:25064
```text
25063: 			if (empty($filtered_data['id_tariff'])) return $this->showError('404', 'error', 'empty tariff');
25064: 			if ($this->id_role != 4) 
25065: 			{
```

### archive_17012026_1259/taxi/models/api.php:25381
```text
25380: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'account not found');
25381: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25382: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
```

### archive_17012026_1259/taxi/models/api.php:25625
```text
25624: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'currency account not found');
25625: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25626: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong account currency');
```

### archive_17012026_1259/taxi/models/api.php:25845
```text
25844: 				{
25845: 					if ($this->id_role != 4 && $data['sender'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
25846: 				}
```

### archive_17012026_1259/taxi/models/api.php:25888
```text
25887: 				if (empty($d['id_currency_account'])) return $this->showError('404', 'error', 'from account not found');
25888: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'from: not enough rights');
25889: 				if ($d['currency'] != $filtered_data['currency']) return $this->showError('404', 'error', 'wrong from account currency');
```

### archive_17012026_1259/taxi/models/api.php:26437
```text
26436: 			if ($d['id_deal_status'] != 3) return $this->showError('404', 'error', 'wrong d_status');
26437: 			if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
26438: 
```

### archive_17012026_1259/taxi/models/api.php:26613
```text
26612: 			if ($d['id_deal_status'] != 3 && $d['id_deal_status'] != 1) return $this->showError('404', 'error', 'wrong d_status');
26613: 			if ($this->id_role != 4 && ($d['customer'] != $_SESSION[UID] || $d['performer'] != $_SESSION[UID])) return $this->showError('404', 'error', 'user not customer or performer');
26614: 
```

### archive_17012026_1259/taxi/models/api.php:26784
```text
26783: 			{
26784: 				if ($this->id_role != 4 && $d['performer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not performer');
26785: 			}
```

### archive_17012026_1259/taxi/models/api.php:26788
```text
26787: 			{
26788: 				if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
26789: 			}
```

## 3. Предварительная доказанная матрица

| Operation | Allowed role IDs | Explicit rejection | Confidence |
|---|---|---|---|

| `registerUser` | `[4]` | not established | `CONFIRMED` |

| `registerUser` | `[4]` | not established | `CONFIRMED` |

| `selectUser` | `[4]` | not established | `CONFIRMED` |

| `selectUser` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `createOrder` | `[1, 4, 5]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `selectActiveOrder` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `unknown` | `[1, 5]` | not established | `CONFIRMED` |

| `selectProcessingOrder` | `[2, 4]` | not established | `CONFIRMED` |

| `selectArchiveOrder` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `unknown` | `[1, 5]` | not established | `CONFIRMED` |

| `setDriver` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `setCarIsArrived` | `[2]` | not established | `CONFIRMED` |

| `completeOrder` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `cancelOrder` | `[1, 2, 4, 5]` | not established | `CONFIRMED` |

| `cancelOrder` | `[4]` | not established | `CONFIRMED` |

| `rateOrder` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `selectOrder` | `[1, 2, 4, 5]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `confirmOrder` | `[1, 5]` | not established | `CONFIRMED` |

| `startOrder` | `[2]` | not established | `CONFIRMED` |

| `editWaitingTime` | `[1, 5]` | not established | `CONFIRMED` |

| `setCarUsed` | `[2]` | not established | `CONFIRMED` |

| `editOrder` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `selectFavoriteUser` | `[4]` | not established | `CONFIRMED` |

| `selectFavoriteUser` | `[4]` | not established | `CONFIRMED` |

| `addFavoriteUser` | `[4]` | not established | `CONFIRMED` |

| `removeFavoriteUser` | `[4]` | not established | `CONFIRMED` |

| `selectReferralUrl` | `[4]` | not established | `CONFIRMED` |

| `selectReferralUser` | `[4]` | not established | `CONFIRMED` |

| `selectReferralUser` | `[4]` | not established | `CONFIRMED` |

| `setOrderTips` | `[1, 2, 5]` | not established | `CONFIRMED` |

| `createTrip` | `[2]` | not established | `CONFIRMED` |

| `createTrip` | `[4]` | yes | `CONFIRMED` |

| `createTrip` | `[4]` | not established | `CONFIRMED` |

| `selectTrip` | `[1, 2, 4, 5]` | not established | `CONFIRMED` |

| `unknown` | `[2, 4]` | not established | `CONFIRMED` |

| `editTrip` | `[2, 4]` | not established | `CONFIRMED` |

| `offerOrder` | `[1, 5]` | not established | `CONFIRMED` |

| `createDropboxFile` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `selectDataPrivate` | `[4]` | not established | `CONFIRMED` |

| `selectCart` | `[4]` | not established | `CONFIRMED` |

| `selectCart` | `[2, 4]` | not established | `CONFIRMED` |

| `queryString` | `[4]` | not established | `CONFIRMED` |

| `selectCartBlock` | `[2, 4]` | not established | `CONFIRMED` |

| `selectCartBlock` | `[4]` | not established | `CONFIRMED` |

| `selectCartBlock` | `[2, 4]` | not established | `CONFIRMED` |

| `selectCartBlock` | `[4]` | not established | `CONFIRMED` |

| `editTicket` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `translate` | `[4]` | not established | `CONFIRMED` |

| `statusCartBlock` | `[4]` | not established | `CONFIRMED` |

| `statusCartBlock` | `[2]` | yes | `CONFIRMED` |

| `statusCartBlock` | `[4]` | yes | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `selectInnerUser` | `[4]` | not established | `CONFIRMED` |

| `selectInnerUser` | `[4]` | not established | `CONFIRMED` |

| `requestTemplate` | `[4]` | not established | `CONFIRMED` |

| `selectContact` | `[4]` | not established | `CONFIRMED` |

| `selectContact` | `[4]` | not established | `CONFIRMED` |

| `selectContact` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `createTask` | `[4]` | not established | `CONFIRMED` |

| `controlTask` | `[4]` | not established | `CONFIRMED` |

| `selectTask` | `[4]` | not established | `CONFIRMED` |

| `addTaskLog` | `[4]` | not established | `CONFIRMED` |

| `getTaskLog` | `[4]` | not established | `CONFIRMED` |

| `setTaskStatus` | `[4]` | not established | `CONFIRMED` |

| `checkTicket` | `[4, 6]` | not established | `CONFIRMED` |

| `selectCheckTicketLog` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `selectPayment` | `[4]` | not established | `CONFIRMED` |

| `unknown` | `[4]` | not established | `CONFIRMED` |

| `confirmDeal` | `[4]` | not established | `CONFIRMED` |

| `confirmDeal` | `[4]` | not established | `CONFIRMED` |


## 4. Как читать матрицу

Если код содержит `id_role != 2 && id_role != 4`, это доказывает, что операция допускает роли `2` и `4` относительно данной проверки. Это не доказывает, что эти роли являются единственными глобально разрешёнными ролями для всей системы. Role policy может иметь дополнительные checks на другом уровне.

Поэтому текущая матрица является **operation-local**, а не глобальной моделью Authorization.

## 5. Что пока нельзя утверждать


Нельзя автоматически записывать:

```text
Role 2 = Driver
Role 4 = Admin
Role 5 = ...
```

если соответствующий mapping не найден в source/config.

Нельзя считать отсутствие проверки `id_role` доказательством `ALLOW_ALL`.

Нельзя превращать список ролей в глобальную permission matrix без обхода всех relevant branches.

Нельзя считать `user_roles` полной authorization policy только из его названия.

## 6. Следующий gap


Остаётся собрать полный набор role-sensitive operations:

```text
$this->id_role
taxi::$id_role
u_a_role
user_roles
query_roles
wrong user role
permission / access denied
```

После этого можно построить:

```text
Role ID
   ×
Operation
   ↓
ALLOW / REJECT / UNKNOWN
```

и отдельно, при наличии source evidence:

```text
Role ID
   ↓
Business Role Name
```

## 7. Методологический результат


RP-06 подтверждает важное ограничение v2.3:

> Authorization matrix строится из проверок, а не из ролей.

То есть:

```text
code check
    ↓
Evidence
    ↓
Claim
    ↓
operation-local authorization relation
```

а не:

```text
role name
    ↓
assumed permissions
```

Новый MCR не требуется.

## 8. Gap Report

```text
G-06-01
Полная role × operation matrix
STATUS: OPEN

G-06-02
Mapping role ID → business role
STATUS: OPEN

G-06-03
query_roles semantics
STATUS: OPEN

G-06-04
role-specific config role{N}.php
STATUS: OPEN
```

## 9. Следующий шаг

Следующий проход должен расширить поиск не по всему backend без ограничений, а по конкретным role-sensitive constructs:

```text
$this->id_role
taxi::$id_role
user_roles
query_roles
wrong user role
```

с целью закрыть G-06-01.

После этого можно будет сопоставить полученную матрицу с конкретными Frontend clients и увидеть, какие role-dependent capabilities реально используются каждым клиентом.
