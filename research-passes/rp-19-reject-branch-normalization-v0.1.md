# Backend Semantic Graph — Research Pass 19
# Reject Branch Normalization v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-18 Authorization Control-Flow Normalization v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Цель
Довести несколько наиболее явных role-sensitive reject branches до semantic decision и проверить, повторяется ли общий authorization pattern.

## 2. Критерий

```text
Role condition
   ↓
branch
   ↓
explicit rejection / continuation
   ↓
semantic decision
```

Только explicit rejection/allow branch становится `CONFIRMED` decision.

## 3. Разобранные reject contexts
### `registerUser` — `archive_17012026_1259/taxi/models/api.php:37`
```php
				}
				else
				{
					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
					$sql_user = "'" . $_SESSION[UID] . "'";
				}
			}
			else
			{
				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
				$sql_user = "'" . $_SESSION[UID] . "'";
			}

			if (empty($phone) && empty($email) && empty($tg) && empty($wa)) 
			{
				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
			} 
			else
			{
				$sql_email = "`email` = '" . $email . "'";
				$sql_phone = "`phone` = '" . preg_replace('/[^0-9]+/','',$phone) . "'";
				$sql_tg = "`tg` = '" . $tg . "'";
				$sql_wa = "`wa` = '" . $wa . "'";				
				
				$sql_field = array();
				$sql_where = array();
				if (empty($phone)) 
				{
					$sql_phone = "`phone` = NULL";
```

### `editUser` — `archive_17012026_1259/taxi/models/api.php:1056`
```php
					case '6':
						$allowed_fields = array(
						);
						break;
					default:
						return $this->showError('404', 'error', 'wrong user role');
				}
			}
			else
			{
				if ($this->id_role != 4) 
				{
					return $this->showError('404', 'error', 'not enough rights');
				}
				
				$s = "SELECT 
						`id_role`,
						`id_user`,
						`phone`,
						`email`,
						`id_verification_status`,
						`tg`,
						`wa`
					FROM `users`
					WHERE 
						`id_user` = '" . $id_user . "'
					LIMIT 1
					";

				$q = query($s);
				if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

### `controlCar` — `archive_17012026_1259/taxi/models/api.php:2200`
```php
					$car = $this->createCar($filtered_data,$id_user);
					if (!empty($car['error'])) 
					{
						return $this->showError('404', 'error', $car['error']);
					}
					$user = $id_user;
				}
			}
			else
			{			
				if (!empty($id_user) && $this->id_role != 4) 
				{
					return $this->showError('404', 'error', 'not enough rights for assign');
				}
				
				$s = "SELECT
						`car`.`id_car`,
						`car`.`license_plate`,
						GROUP_CONCAT(`car_users`.`id_user` SEPARATOR ',') as u_id
					FROM `car`
					LEFT JOIN `car_users` USING (`id_car`)
					WHERE
						`car`.`id_car` = '" . $id_car . "'
					GROUP BY
						`car_users`.`id_car`
					";

				$q = query($s);
				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
				$d = fetch_assoc($q);

```

### `createOrder` — `archive_17012026_1259/taxi/models/api.php:3990`
```php
				}
				$add_s .= ",`datetime_start_plan` = '" . $datetime_start_plan . "'";
				unset($filtered_data['datetime_start_plan']);
			}
			if (empty($filtered_data['id_payment_method']))
			{
				return $this->showError('404', 'error', 'empty payment way');
			}
			elseif ($filtered_data['id_payment_method'] == 4)
			{
				if ($this->id_role != 4) return $this->showError('404', 'error', 'forbidden payment way');
			}

			if (!empty($affected_keys['b_options']))
			{				
				unset($data['b_options'][':public']);
				unset($data['b_options'][':u_id_alias']);
				if (!empty($this->constant['b_options_valid_keys']))
				{
					if (!is_array($data['b_options'])) return $this->showError('404', 'error', 'b_options not array');
					$wrong_b_options = array();
					foreach($data['b_options'] as $key => $value)
					{
						if (empty($this->constant['b_options_valid_keys'][$key])) $wrong_b_options[] = $key;
					}
					if (!empty($wrong_b_options)) return $this->showError('404', 'error', 'wrong b_options keys: ' . implode(",", $wrong_b_options));
				}
				$filtered_data['options'] = real_escape_string(json_encode($data['b_options']));
			}		
			if (!empty($affected_keys['b_contact']))
			{
```

### `selectActiveOrder` — `archive_17012026_1259/taxi/models/api.php:4481`
```php
				'data' 		=>	$out
			);
		}

		public function selectActiveOrder($fields = 0)
		{
			if (empty($_SESSION[UID])) {
				return $this->showError('404', 'error', 'unauthorized access');
			}

			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
			{
				return $this->showError('404', 'error', 'wrong user role');
			}

			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = $sql_where = '';
						
			$field_flag = array();
			if (!empty($fields))
			{
				$field_arr	= get_field_arr('activeOrder',$this->id_role);
				$bin_arr = get_bin_arr();

				foreach(str_split($fields) as $index => $char)
				{
					$value = get_0_64($char);
					if (empty($value)) continue;
					foreach($bin_arr as $bin_i)
					{
						if ($value & $bin_i) 
						{
```

## 4. Нормализация

### `registerUser`
- Role IDs in branch: `4=Administrator`
- Explicit rejection evidence: `if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');`
- Decision status: `CONFIRMED` where the branch explicitly rejects access.

### `editUser`
- Role IDs in branch: `4=Administrator`
- Explicit rejection evidence: `return $this->showError('404', 'error', 'wrong user role');`
- Decision status: `CONFIRMED` where the branch explicitly rejects access.

### `controlCar`
- Role IDs in branch: `4=Administrator`
- Explicit rejection evidence: `return $this->showError('404', 'error', 'not enough rights for assign');`
- Decision status: `CONFIRMED` where the branch explicitly rejects access.

### `createOrder`
- Role IDs in branch: `4=Administrator`
- Explicit rejection evidence: `if ($this->id_role != 4) return $this->showError('404', 'error', 'forbidden payment way');`
- Decision status: `CONFIRMED` where the branch explicitly rejects access.

### `selectActiveOrder`
- Role IDs in branch: `1=Client, 2=Driver, 5=Agent`
- Explicit rejection evidence: `return $this->showError('404', 'error', 'unauthorized access');`
- Decision status: `CONFIRMED` where the branch explicitly rejects access.

## 5. Общий паттерн

В разобранных ветвях повторяется форма:

```text
authenticated API context
        ↓
runtime id_role
        ↓
role condition
        ↓
explicit reject branch
        ↓
return error
```

Это позволяет выделить устойчивый authorization pattern, но пока не доказывает наличие одного общего helper, реализующего все проверки.

## 6. Важное различие

Не все role-sensitive branches являются одинаковыми:

```text
ROLE GATE
    role not allowed → reject

ROLE + PRECONDITION
    role allowed
       + verification/state/hash condition
       → allow/reject

QUERY ROLE SET
    role ∉ query_roles → reject
```

Поэтому в Semantic Graph нельзя сводить все проверки к одной relation `CAN_EXECUTE` без сохранения Preconditions.

## 7. Текущий semantic layer

| Mechanism | Semantic type | Status |
|---|---|---|
| direct role rejection | AUTHORIZATION | CONFIRMED |
| role + verification/state | AUTHORIZATION + PRECONDITION | CONFIRMED where branch explicit |
| `query_roles` membership | AUTHORIZATION | CONFIRMED |

## 8. Gap Report

```text
G-19-01  normalize remaining role-sensitive functions      OPEN
G-19-02  identify shared authorization helper              OPEN
G-19-03  complete Role × Operation matrix                   OPEN
G-19-04  map frontend-consumed authorization capabilities   OPEN
```

## 9. MCR

`MCR = NO CHANGE`.

Общий authorization pattern не требует нового фундаментального типа графа. Preconditions должны оставаться отдельными Claims/relations.

## 10. Следующий шаг

Следующий проход должен проверить, являются ли разобранные role checks вызовами одного общего helper (`check_auth_user`, role validation, permission helper и т.п.) или независимыми локальными gates. Если общий helper найден, исследовать его как потенциальную Authorization implementation, не объявляя его автоматически domain Capability.