# Backend Semantic Graph — Research Pass 35
# Taxi Authentication Frontend ↔ Core Backend Trace v0.2

**Статус:** EVIDENCE-GROUNDED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-34 Taxi Frontend Login Endpoint Trace v0.1  
**Источники:** `taxi-master.zip` + `archive_17012026_1259_clear.zip`

## 1. Research Question

> Можно ли сопоставить конкретный authentication call Taxi Frontend с конкретным Core Backend handler и проследить credential/value-flow до `check_auth_user()`?

## 2. Frontend auth contexts

### `src/API/car.ts:95`
```text
83:       b_l_c: [{
84:         location: locationClass.id,
85:         value: country,
86:       }],
87:     }],
88:   }) })
89:   await axios.post(`${Config.API_URL}/car/${id}`, formData)
90: })
91: 
92: const _getUserCars = (
93:   { formData }: IApiMethodArguments,
94: ): Promise<any> => {
95:   return axios.post(`${Config.API_URL}/user/authorized/car`, formData)
96:     .then(res => Object.values(res?.data?.data?.car || {}))
97: }
98: export const getUserCars = apiMethod<typeof _getUserCars>(_getUserCars)
99: 
100: const _getUserCar = (
101:   { formData }: IApiMethodArguments,
102:   id: IUser['u_id'],
103: ): Promise<ICar | null> => {
104:   addToFormData(formData, {
105:     array_type: 'list',
106:   })
107: 
108:   return axios.post(`${Config.API_URL}/user/${id}/car`, formData)
109:     .then(res => res.data.data)
110:     .then(res => (res.car && res.car[0]) || null)
111: }
112: export const getUserCar = apiMethod<typeof _getUserCar>(_getUserCar)
113: 
114: export const editCar = apiMethod(async(
115:   { formData }: IApiMethodArguments,
116:   id: ICar['c_id'],
117:   fields: Partial<Pick<ICar,
118:     'cm_id' |
119:     'seats' |
120:     'registration_plate' |
```

### `src/API/car.ts:184`
```text
172:     await axios.post(`${Config.API_URL}/user`, formData)
173:   if (
174:     response.code === '404' &&
175:     response.message === 'user or modified data not found'
176:   ) response.data = { detail: 'not_modified' }
177:   return response
178: })
179: 
180: async function _getUserDrivenCar(
181:   { formData }: IApiMethodArguments,
182: ): Promise<ICar> {
183:   const { data } = await axios.post(
184:     `${Config.API_URL}/user/authorized/car/driven`,
185:     formData,
186:   )
187:   return Object.values(data.data.car)[0] as ICar
188: }
189: export const getUserDrivenCar =
190:   apiMethod<typeof _getUserDrivenCar>(_getUserDrivenCar)
```

### `src/API/user.ts:27`
```text
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
45:   'u_lang' |
46:   'u_currency' |
47:   'ref_code' |
48:   'u_details'
49: >>
50: export type TEditDriverCheckRequired = TEditUser & Partial<Pick<IUser,
51:   'u_role' |
52:   'u_name' |
```

### `src/API/user.ts:30`
```text
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
45:   'u_lang' |
46:   'u_currency' |
47:   'ref_code' |
48:   'u_details'
49: >>
50: export type TEditDriverCheckRequired = TEditUser & Partial<Pick<IUser,
51:   'u_role' |
52:   'u_name' |
53:   'u_family' |
54:   'u_middle' |
55:   'u_phone' |
```

### `src/API/user.ts:34`
```text
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
45:   'u_lang' |
46:   'u_currency' |
47:   'ref_code' |
48:   'u_details'
49: >>
50: export type TEditDriverCheckRequired = TEditUser & Partial<Pick<IUser,
51:   'u_role' |
52:   'u_name' |
53:   'u_family' |
54:   'u_middle' |
55:   'u_phone' |
56:   'u_email' |
57:   'u_photo' |
58:   'u_city' |
59:   'u_lang_skills' |
```

### `src/API/user.ts:36`
```text
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
45:   'u_lang' |
46:   'u_currency' |
47:   'ref_code' |
48:   'u_details'
49: >>
50: export type TEditDriverCheckRequired = TEditUser & Partial<Pick<IUser,
51:   'u_role' |
52:   'u_name' |
53:   'u_family' |
54:   'u_middle' |
55:   'u_phone' |
56:   'u_email' |
57:   'u_photo' |
58:   'u_city' |
59:   'u_lang_skills' |
60:   'u_description' |
61:   'u_birthday' |
```

### `src/API/user.ts:93`
```text
81:   'ref_code' |
82:   'u_details'
83: >>
84: 
85: const _editUser = (
86:   { formData }: IApiMethodArguments,
87:   data:
88:     TEditClient |
89:     TEditDriverCheckRequired |
90:     TEditDriverCheckActive,
91: ) => {
92:   // @TODO вернуть u_city когда наладим автозаполнение
93:   const { token, u_hash, u_id, u_city, ...userData } = data as (
94:     TEditClient &
95:     TEditDriverCheckRequired &
96:     TEditDriverCheckActive
97:   )
98:   if (token && u_hash && u_id) addToFormData(formData, { token, u_hash, u_id })
99:   addToFormData(formData, {
100:     data: JSON.stringify({
101:       u_city: u_city || undefined,
102:       ...reverseConvertUser(userData),
103:     }),
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
109: export const editUser = apiMethod<typeof _editUser>(_editUser)
110: export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })
```

### `src/API/user.ts:98`
```text
86:   { formData }: IApiMethodArguments,
87:   data:
88:     TEditClient |
89:     TEditDriverCheckRequired |
90:     TEditDriverCheckActive,
91: ) => {
92:   // @TODO вернуть u_city когда наладим автозаполнение
93:   const { token, u_hash, u_id, u_city, ...userData } = data as (
94:     TEditClient &
95:     TEditDriverCheckRequired &
96:     TEditDriverCheckActive
97:   )
98:   if (token && u_hash && u_id) addToFormData(formData, { token, u_hash, u_id })
99:   addToFormData(formData, {
100:     data: JSON.stringify({
101:       u_city: u_city || undefined,
102:       ...reverseConvertUser(userData),
103:     }),
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
109: export const editUser = apiMethod<typeof _editUser>(_editUser)
110: export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })
```

### `src/API/user.ts:110`
```text
98:   if (token && u_hash && u_id) addToFormData(formData, { token, u_hash, u_id })
99:   addToFormData(formData, {
100:     data: JSON.stringify({
101:       u_city: u_city || undefined,
102:       ...reverseConvertUser(userData),
103:     }),
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
109: export const editUser = apiMethod<typeof _editUser>(_editUser)
110: export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })
```

### `src/API/auth.ts:2`
```text
1: import axios from 'axios'
2: import { EUserRoles, ITokens, IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: import { ERegistrationType } from '../state/user/constants'
7: 
8: const _register = (
9:   { formData }: IApiMethodArguments,
10:   data: Partial<IUser>,
11: ): Promise<{
12:   u_id: IUser['u_id'],
13:   email_status: boolean,
14:   string: string,
15:   error?: string
16: } | null> => {
17:   addToFormData(formData, reverseConvertUser(data))
18:   if (data.u_role === EUserRoles.Driver) addToFormData(formData, { 'st': 1 })
19:   return axios.post(`${Config.API_URL}/register`, formData)
20:     .then(res => res.data)
21:     .then(res => {
22:       if (res.status === 'error') {
23:         return {
24:           u_id: null,
25:           email_status: false,
26:           string: res.data,
27:           error: res.message,
```

### `src/API/auth.ts:37`
```text
25:           email_status: false,
26:           string: res.data,
27:           error: res.message,
28:         }
29:       }
30:       return res.data
31:     })
32: }
33: /**
34:  * @returns email_status - if email is specified
35:  * @returns string - password if email is not specified
36:  */
37: export const register = apiMethod<typeof _register>(_register, { authRequired: false })
38: 
39: const _remindPassword = (
40:   { formData }: IApiMethodArguments,
41:   email: IUser['u_email'],
42: ) => {
43:   addToFormData(formData, {
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
```

### `src/API/auth.ts:51`
```text
39: const _remindPassword = (
40:   { formData }: IApiMethodArguments,
41:   email: IUser['u_email'],
42: ) => {
43:   addToFormData(formData, {
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
```

### `src/API/auth.ts:53`
```text
41:   email: IUser['u_email'],
42: ) => {
43:   addToFormData(formData, {
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
```

### `src/API/auth.ts:56`
```text
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
```

### `src/API/auth.ts:60`
```text
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
```

### `src/API/auth.ts:66`
```text
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
```

### `src/API/auth.ts:69`
```text
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
```

### `src/API/auth.ts:72`
```text
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
```

### `src/API/auth.ts:79`
```text
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
```

### `src/API/auth.ts:86`
```text
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
```

### `src/API/auth.ts:93`
```text
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
```

### `src/API/auth.ts:98`
```text
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
```

### `src/API/auth.ts:101`
```text
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
```

### `src/API/auth.ts:102`
```text
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
```

### `src/API/auth.ts:103`
```text
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
```

### `src/API/auth.ts:105`
```text
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
```

### `src/API/auth.ts:106`
```text
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
```

### `src/API/auth.ts:107`
```text
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
```

### `src/API/auth.ts:108`
```text
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
```

### `src/API/auth.ts:109`
```text
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
```

### `src/API/auth.ts:110`
```text
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
```

### `src/API/auth.ts:111`
```text
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
```

### `src/API/auth.ts:117`
```text
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
```

### `src/API/auth.ts:122`
```text
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
```

### `src/API/auth.ts:129`
```text
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
```

### `src/API/auth.ts:140`
```text
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
```

### `src/API/auth.ts:142`
```text
130:     type: ERegistrationType.Whatsapp,
131:     u_role: EUserRoles.Client,
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
```

### `src/API/auth.ts:144`
```text
132:   })
133:   return axios.post(`${Config.API_URL}/register`, waData)
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
```

### `src/API/auth.ts:155`
```text
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
```

### `src/API/auth.ts:157`
```text
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
```

### `src/API/auth.ts:158`
```text
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
```

### `src/API/auth.ts:159`
```text
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
```

### `src/API/auth.ts:161`
```text
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
```

### `src/API/auth.ts:166`
```text
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
```

### `src/API/auth.ts:169`
```text
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
```

### `src/API/auth.ts:170`
```text
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
```

### `src/API/auth.ts:171`
```text
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
```

### `src/API/auth.ts:172`
```text
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
```

### `src/API/auth.ts:174`
```text
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
```

### `src/API/auth.ts:178`
```text
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
```

### `src/API/auth.ts:179`
```text
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
```

### `src/API/auth.ts:180`
```text
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
```

### `src/API/auth.ts:181`
```text
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
```

### `src/API/auth.ts:188`
```text
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:189`
```text
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:190`
```text
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:191`
```text
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:193`
```text
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:194`
```text
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:195`
```text
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:196`
```text
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:202`
```text
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:209`
```text
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/index.ts:33`
```text
21: import SITE_CONSTANTS from '../siteConstants'
22: import Config from '../config'
23: import store from '../state'
24: import { configSelectors } from '../state/config'
25: import { userSelectors } from '../state/user'
26: import { EBookingActions } from './constants'
27: import { getCacheVersion } from './cacheVersion'
28: 
29: export { getCacheVersion, EBookingActions }
30: export {
31:   register,
32:   remindPassword,
33:   login,
34:   whatsappSignUp,
35:   googleLogin,
36:   logout,
37: } from './auth'
38: export {
39:   getUser,
40:   getUsers,
41:   getAuthorizedUser,
42:   editUser,
43:   editUserAfterRegister,
44: } from './user'
45: export {
46:   createCar,
47:   createUserCar,
48:   editCar,
49:   getUserCars,
50:   getUserCar,
51:   driveCar,
52:   getUserDrivenCar,
53:   getCar,
54:   getCars,
55: } from './car'
56: export {
57:   postDrive,
58:   cancelDrive,
```

### `src/API/index.ts:35`
```text
23: import store from '../state'
24: import { configSelectors } from '../state/config'
25: import { userSelectors } from '../state/user'
26: import { EBookingActions } from './constants'
27: import { getCacheVersion } from './cacheVersion'
28: 
29: export { getCacheVersion, EBookingActions }
30: export {
31:   register,
32:   remindPassword,
33:   login,
34:   whatsappSignUp,
35:   googleLogin,
36:   logout,
37: } from './auth'
38: export {
39:   getUser,
40:   getUsers,
41:   getAuthorizedUser,
42:   editUser,
43:   editUserAfterRegister,
44: } from './user'
45: export {
46:   createCar,
47:   createUserCar,
48:   editCar,
49:   getUserCars,
50:   getUserCar,
51:   driveCar,
52:   getUserDrivenCar,
53:   getCar,
54:   getCars,
55: } from './car'
56: export {
57:   postDrive,
58:   cancelDrive,
59:   getOrders,
60:   getOrder,
```

### `src/API/index.ts:37`
```text
25: import { userSelectors } from '../state/user'
26: import { EBookingActions } from './constants'
27: import { getCacheVersion } from './cacheVersion'
28: 
29: export { getCacheVersion, EBookingActions }
30: export {
31:   register,
32:   remindPassword,
33:   login,
34:   whatsappSignUp,
35:   googleLogin,
36:   logout,
37: } from './auth'
38: export {
39:   getUser,
40:   getUsers,
41:   getAuthorizedUser,
42:   editUser,
43:   editUserAfterRegister,
44: } from './user'
45: export {
46:   createCar,
47:   createUserCar,
48:   editCar,
49:   getUserCars,
50:   getUserCar,
51:   driveCar,
52:   getUserDrivenCar,
53:   getCar,
54:   getCars,
55: } from './car'
56: export {
57:   postDrive,
58:   cancelDrive,
59:   getOrders,
60:   getOrder,
61:   editOrder,
62:   takeOrder,
```

### `src/API/index.ts:41`
```text
29: export { getCacheVersion, EBookingActions }
30: export {
31:   register,
32:   remindPassword,
33:   login,
34:   whatsappSignUp,
35:   googleLogin,
36:   logout,
37: } from './auth'
38: export {
39:   getUser,
40:   getUsers,
41:   getAuthorizedUser,
42:   editUser,
43:   editUserAfterRegister,
44: } from './user'
45: export {
46:   createCar,
47:   createUserCar,
48:   editCar,
49:   getUserCars,
50:   getUserCar,
51:   driveCar,
52:   getUserDrivenCar,
53:   getCar,
54:   getCars,
55: } from './car'
56: export {
57:   postDrive,
58:   cancelDrive,
59:   getOrders,
60:   getOrder,
61:   editOrder,
62:   takeOrder,
63:   chooseCandidate,
64:   setOrderState,
65:   setOrderRating,
66:   setWaitingTime,
```

### `src/API/index.ts:85`
```text
73:   getPolygonsIdsOnPoint,
74: } from './polygon'
75: 
76: const _uploadFile = (
77:   { formData }: IApiMethodArguments,
78:   data: any,
79: ): Promise<{
80:   dl_id: string
81: } | null> => {
82:   return getBase64(data.file)
83:     .then(base64 => {
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
88:         private: 0,
89:       })
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
```

### `src/API/index.ts:86`
```text
74: } from './polygon'
75: 
76: const _uploadFile = (
77:   { formData }: IApiMethodArguments,
78:   data: any,
79: ): Promise<{
80:   dl_id: string
81: } | null> => {
82:   return getBase64(data.file)
83:     .then(base64 => {
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
88:         private: 0,
89:       })
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
```

### `src/API/index.ts:96`
```text
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
88:         private: 0,
89:       })
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
```

### `src/API/index.ts:108`
```text
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
122:     t_id: ITrip['t_id'],
123: }> => {
124:   addToFormData(formData, {
125:     data: JSON.stringify(reverseConvertTrip(data)),
126:   })
127: 
128:   return axios.post(`${Config.API_URL}/trip`, formData)
129:     .then(res => res.data)
130: }
131: export const postTrip = apiMethod<typeof _postTrip>(_postTrip)
132: 
133: const _getTrips = (
```

### `src/API/index.ts:116`
```text
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
122:     t_id: ITrip['t_id'],
123: }> => {
124:   addToFormData(formData, {
125:     data: JSON.stringify(reverseConvertTrip(data)),
126:   })
127: 
128:   return axios.post(`${Config.API_URL}/trip`, formData)
129:     .then(res => res.data)
130: }
131: export const postTrip = apiMethod<typeof _postTrip>(_postTrip)
132: 
133: const _getTrips = (
134:   { formData }: IApiMethodArguments,
135:   type: EOrderTypes = EOrderTypes.Active,
136: ): Promise<IOrder[]> => {
137:   addToFormData(formData, {
138:     array_type: 'list',
139:   })
140: 
141:   return axios.post(`${Config.API_URL}/trip`, formData)
```

### `src/API/index.ts:175`
```text
163:   return axios.post(`${Config.API_URL}/trip/get`, formData)
164:     .then(res => res.data)
165:     .then(res =>
166:       res.data.trip || [],
167:     )
168:     .then(res => res.map((item: any) => convertTrip(item)))
169:     .then(res =>
170:       res.sort(
171:         (a: IOrder, b: IOrder) => a.b_start_datetime < b.b_start_datetime ? 1 : -1,
172:       ),
173:     )
174: }
175: export const getWashTrips = apiMethod<typeof _getWashTrips>(_getWashTrips, { authRequired: false })
176: 
177: const _getImageBlob = (
178:   { formData }: IApiMethodArguments,
179:   id: number,
180: ) => {
181:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
182:     responseType: 'blob',
183:   }).then(res => {
184:     return [id, URL.createObjectURL(res.data)]
185:   })
186: }
187: export const getImageBlob = apiMethod<typeof _getImageBlob>(_getImageBlob)
188: 
189: const _getImageFile = (
190:   { formData }: IApiMethodArguments,
191:   id: number,
192: ): Promise<[number, File]> => {
193:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
194:     responseType: 'blob',
195:   }).then(res => {
196:     return [id, new File([res.data], String(id))]
197:   })
198: }
199: export const getImageFile = apiMethod<typeof _getImageFile>(_getImageFile)
200: 
```

### `src/API/index.ts:288`
```text
276:         limit: 1,
277:         format: 'json',
278:         'accept-language': language.iso,
279:       },
280:     },
281:   )
282:     .then(res =>
283:       res.data[0] &&
284:             ({ ...res.data[0], lat: parseFloat(res.data[0].lat), lon: parseFloat(res.data[0].lon) }),
285:     )
286: }
287: 
288: const orsToken = '<REDACTED>'
289: 
290: export const makeRoutePoints = (from: IAddressPoint, to: IAddressPoint): Promise<IRouteInfo> => {
291:   const convertPoint = (point: IAddressPoint) => `${point.longitude},${point.latitude}`
292: 
293:   return axios.get(
294:     'https://api.openrouteservice.org/v2/directions/driving-car',
295:     {
296:       params: {
297:         api_key: orsToken,
298:         start: convertPoint(from),
299:         end: convertPoint(to),
300:       },
301:     },
302:   )
303:     .then(res => res.data)
304:     .then(res => {
305:       const hours = Math.floor(res.features[0].properties.summary.duration / (60 * 60))
306:       const minutes = Math.round((res.features[0].properties.summary.duration - (hours * 60 * 60)) / 60)
307:       return {
308:         distance: parseFloat((res.features[0].properties.summary.distance / 1000).toFixed(2)),
309:         time: {
310:           hours,
311:           minutes,
312:         },
313:         points: res.features[0].geometry
```

### `src/API/index.ts:297`
```text
285:     )
286: }
287: 
288: const orsToken = '<REDACTED>'
289: 
290: export const makeRoutePoints = (from: IAddressPoint, to: IAddressPoint): Promise<IRouteInfo> => {
291:   const convertPoint = (point: IAddressPoint) => `${point.longitude},${point.latitude}`
292: 
293:   return axios.get(
294:     'https://api.openrouteservice.org/v2/directions/driving-car',
295:     {
296:       params: {
297:         api_key: orsToken,
298:         start: convertPoint(from),
299:         end: convertPoint(to),
300:       },
301:     },
302:   )
303:     .then(res => res.data)
304:     .then(res => {
305:       const hours = Math.floor(res.features[0].properties.summary.duration / (60 * 60))
306:       const minutes = Math.round((res.features[0].properties.summary.duration - (hours * 60 * 60)) / 60)
307:       return {
308:         distance: parseFloat((res.features[0].properties.summary.distance / 1000).toFixed(2)),
309:         time: {
310:           hours,
311:           minutes,
312:         },
313:         points: res.features[0].geometry
314:           .coordinates.map((item: [number, number]) => [item[1], item[0]]),
315:       }
316:     })
317: }
318: 
319: export const notifyPosition = (point: IAddressPoint) => {
320:   const userID = userSelectors.user(store.getState())?.u_id
321: 
322:   axios.post('http://jecat.ru/car_api/api/notifypos.php', {
```

## 3. Core Backend auth contexts

### `archive_17012026_1259/taxi/controllers/c_api.php:26`
```php
16: 		$API->tuneLimit(isset($_REQUEST['lo'])?$_REQUEST['lo']:NULL,isset($_REQUEST['lc'])?$_REQUEST['lc']:NULL);
17: 		if (isset($_REQUEST['appUrl']))
18: 		{
19: 			$API->appUrl = $_REQUEST['appUrl'];
20: 		}
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
```

### `archive_17012026_1259/taxi/controllers/c_api.php:27`
```php
17: 		if (isset($_REQUEST['appUrl']))
18: 		{
19: 			$API->appUrl = $_REQUEST['appUrl'];
20: 		}
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
```

### `archive_17012026_1259/taxi/controllers/c_api.php:31`
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
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
```

### `archive_17012026_1259/taxi/controllers/c_api.php:40`
```php
30: 			{
31: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
32: 
33: 			}
34: 			elseif ($_GET['par'][1] == 'logout')
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
59: 						$out = $API->selectUser(isset($_GET['par'][2])? trim($_GET['par'][2]):'',isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
60: 					}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:41`
```php
31: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
32: 
33: 			}
34: 			elseif ($_GET['par'][1] == 'logout')
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
59: 						$out = $API->selectUser(isset($_GET['par'][2])? trim($_GET['par'][2]):'',isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
60: 					}
61: 				}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:22`
```php
12: 		if (isset($_REQUEST['array_type']) && trim($_REQUEST['array_type']) == 'list')
13: 		{
14: 			$API->associativeArray = false;
15: 		}
16: 		$API->tuneLimit(isset($_REQUEST['lo'])?$_REQUEST['lo']:NULL,isset($_REQUEST['lc'])?$_REQUEST['lc']:NULL);
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
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:23`
```php
13: 		{
14: 			$API->associativeArray = false;
15: 		}
16: 		$API->tuneLimit(isset($_REQUEST['lo'])?$_REQUEST['lo']:NULL,isset($_REQUEST['lc'])?$_REQUEST['lc']:NULL);
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
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:27`
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
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:36`
```php
26: 			{
27: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
28: 
29: 			}
30: 			elseif ($_GET['par'][1] == 'logout')
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
55: 					}
56: 					else
```

### `archive_17012026_1259/taxi/cache/data.php:987`
```php
977: </head>
978: <body style="margin: 0; padding: 0;">	
979: 	<table align="center" cellpadding="0" cellspacing="0" border="0" style="border-collapse:collapse;margin:0;padding:0;color:#000000;font-family:sans-serif;font-size:14px;" width="100%">
980: 	<tr>
981: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><img src="{$so_data[%5B%22site_images%22%2C%22logo%22%2C%22file%22%5D]}" style="border:none;"></td>
982: 	</tr>
983: 	<tr>
984: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top">Dear, {$u_name} {$u_middle} {$u_family}!</td>
985: 	</tr>
986: 	<tr>
987: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">Thank you for signing up for {$so_data[%5B%22lang_vls%22%2C%22site_name%22%2C%22%7B%24lang%7D%22%5D]}!</td>
988: 	</tr>
989: 	<tr>
990: 		<td align="left" style="padding:15px 6px 0 21px;" valign="top" width="35">Login</td>
991: 		<td align="left" style="padding:15px 21px 0 6px;" valign="top">{$u_email}</td>
992: 	</tr>
993: 	<tr>
994: 		<td align="left" style="padding:5px 6px 0 21px;" valign="top" width="35">Password</td>
995: 		<td align="left" style="padding:5px 21px 0 6px;" valign="top">{$password}</td>
996: 	</tr>
997: 	<tr>
998: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><a href="Find tickets">Find tickets</a></td>
999: 	</tr>
1000: 	<tr>
1001: 		<td colspan="2" align="left" style="padding:37px 21px 0;font-weight:bold;" valign="top">Need help? Contact customer support</td>
1002: 	</tr>
1003: 	<tr>
1004: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"> {$so_data[%5B%22site_constants%22%2C%22site_contact_phone%22%2C%22value%22%5D]}</td>
1005: 	</tr>
1006: 	<tr>
1007: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">{$so_data[%5B%22site_constants%22%2C%22site_contact_email%22%2C%22value%22%5D]}</td>
```

### `archive_17012026_1259/taxi/cache/data.php:990`
```php
980: 	<tr>
981: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><img src="{$so_data[%5B%22site_images%22%2C%22logo%22%2C%22file%22%5D]}" style="border:none;"></td>
982: 	</tr>
983: 	<tr>
984: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top">Dear, {$u_name} {$u_middle} {$u_family}!</td>
985: 	</tr>
986: 	<tr>
987: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">Thank you for signing up for {$so_data[%5B%22lang_vls%22%2C%22site_name%22%2C%22%7B%24lang%7D%22%5D]}!</td>
988: 	</tr>
989: 	<tr>
990: 		<td align="left" style="padding:15px 6px 0 21px;" valign="top" width="35">Login</td>
991: 		<td align="left" style="padding:15px 21px 0 6px;" valign="top">{$u_email}</td>
992: 	</tr>
993: 	<tr>
994: 		<td align="left" style="padding:5px 6px 0 21px;" valign="top" width="35">Password</td>
995: 		<td align="left" style="padding:5px 21px 0 6px;" valign="top">{$password}</td>
996: 	</tr>
997: 	<tr>
998: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><a href="Find tickets">Find tickets</a></td>
999: 	</tr>
1000: 	<tr>
1001: 		<td colspan="2" align="left" style="padding:37px 21px 0;font-weight:bold;" valign="top">Need help? Contact customer support</td>
1002: 	</tr>
1003: 	<tr>
1004: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"> {$so_data[%5B%22site_constants%22%2C%22site_contact_phone%22%2C%22value%22%5D]}</td>
1005: 	</tr>
1006: 	<tr>
1007: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">{$so_data[%5B%22site_constants%22%2C%22site_contact_email%22%2C%22value%22%5D]}</td>
1008: 	</tr>
1009: 	</table>
1010: </body>
```

### `archive_17012026_1259/taxi/cache/data.php:994`
```php
984: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top">Dear, {$u_name} {$u_middle} {$u_family}!</td>
985: 	</tr>
986: 	<tr>
987: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">Thank you for signing up for {$so_data[%5B%22lang_vls%22%2C%22site_name%22%2C%22%7B%24lang%7D%22%5D]}!</td>
988: 	</tr>
989: 	<tr>
990: 		<td align="left" style="padding:15px 6px 0 21px;" valign="top" width="35">Login</td>
991: 		<td align="left" style="padding:15px 21px 0 6px;" valign="top">{$u_email}</td>
992: 	</tr>
993: 	<tr>
994: 		<td align="left" style="padding:5px 6px 0 21px;" valign="top" width="35">Password</td>
995: 		<td align="left" style="padding:5px 21px 0 6px;" valign="top">{$password}</td>
996: 	</tr>
997: 	<tr>
998: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><a href="Find tickets">Find tickets</a></td>
999: 	</tr>
1000: 	<tr>
1001: 		<td colspan="2" align="left" style="padding:37px 21px 0;font-weight:bold;" valign="top">Need help? Contact customer support</td>
1002: 	</tr>
1003: 	<tr>
1004: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"> {$so_data[%5B%22site_constants%22%2C%22site_contact_phone%22%2C%22value%22%5D]}</td>
1005: 	</tr>
1006: 	<tr>
1007: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">{$so_data[%5B%22site_constants%22%2C%22site_contact_email%22%2C%22value%22%5D]}</td>
1008: 	</tr>
1009: 	</table>
1010: </body>
1011: </html>',
1012:     ),
1013:     'email_register_subject' => 
1014:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:995`
```php
985: 	</tr>
986: 	<tr>
987: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">Thank you for signing up for {$so_data[%5B%22lang_vls%22%2C%22site_name%22%2C%22%7B%24lang%7D%22%5D]}!</td>
988: 	</tr>
989: 	<tr>
990: 		<td align="left" style="padding:15px 6px 0 21px;" valign="top" width="35">Login</td>
991: 		<td align="left" style="padding:15px 21px 0 6px;" valign="top">{$u_email}</td>
992: 	</tr>
993: 	<tr>
994: 		<td align="left" style="padding:5px 6px 0 21px;" valign="top" width="35">Password</td>
995: 		<td align="left" style="padding:5px 21px 0 6px;" valign="top">{$password}</td>
996: 	</tr>
997: 	<tr>
998: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"><a href="Find tickets">Find tickets</a></td>
999: 	</tr>
1000: 	<tr>
1001: 		<td colspan="2" align="left" style="padding:37px 21px 0;font-weight:bold;" valign="top">Need help? Contact customer support</td>
1002: 	</tr>
1003: 	<tr>
1004: 		<td colspan="2" align="left" style="padding:17px 21px 0;" valign="top"> {$so_data[%5B%22site_constants%22%2C%22site_contact_phone%22%2C%22value%22%5D]}</td>
1005: 	</tr>
1006: 	<tr>
1007: 		<td colspan="2" align="left" style="padding:5px 21px 0;" valign="top">{$so_data[%5B%22site_constants%22%2C%22site_contact_email%22%2C%22value%22%5D]}</td>
1008: 	</tr>
1009: 	</table>
1010: </body>
1011: </html>',
1012:     ),
1013:     'email_register_subject' => 
1014:     array (
1015:       1 => 'Signing',
```

### `archive_17012026_1259/taxi/cache/data.php:1817`
```php
1807:     ),
1808:     'loading_p' => 
1809:     array (
1810:       1 => 'Погрузка',
1811:       2 => 'Loading',
1812:     ),
1813:     'loading_place_p' => 
1814:     array (
1815:       1 => 'Нас. пункт погрузки',
1816:     ),
1817:     'login' => 
1818:     array (
1819:       1 => 'Логин',
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
```

### `archive_17012026_1259/taxi/cache/data.php:1820`
```php
1810:       1 => 'Погрузка',
1811:       2 => 'Loading',
1812:     ),
1813:     'loading_place_p' => 
1814:     array (
1815:       1 => 'Нас. пункт погрузки',
1816:     ),
1817:     'login' => 
1818:     array (
1819:       1 => 'Логин',
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
1838:     array (
1839:       1 => 'Ошибка выхода',
1840:       2 => 'Logout fail',
```

### `archive_17012026_1259/taxi/cache/data.php:1822`
```php
1812:     ),
1813:     'loading_place_p' => 
1814:     array (
1815:       1 => 'Нас. пункт погрузки',
1816:     ),
1817:     'login' => 
1818:     array (
1819:       1 => 'Логин',
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
1838:     array (
1839:       1 => 'Ошибка выхода',
1840:       2 => 'Logout fail',
1841:     ),
1842:     'logout_success' => 
```

### `archive_17012026_1259/taxi/cache/data.php:1825`
```php
1815:       1 => 'Нас. пункт погрузки',
1816:     ),
1817:     'login' => 
1818:     array (
1819:       1 => 'Логин',
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
1838:     array (
1839:       1 => 'Ошибка выхода',
1840:       2 => 'Logout fail',
1841:     ),
1842:     'logout_success' => 
1843:     array (
1844:       1 => 'Успешный выход',
1845:       2 => 'Logout success',
```

### `archive_17012026_1259/taxi/cache/data.php:1827`
```php
1817:     'login' => 
1818:     array (
1819:       1 => 'Логин',
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
1838:     array (
1839:       1 => 'Ошибка выхода',
1840:       2 => 'Logout fail',
1841:     ),
1842:     'logout_success' => 
1843:     array (
1844:       1 => 'Успешный выход',
1845:       2 => 'Logout success',
1846:     ),
1847:     'loveseat' => 
```

### `archive_17012026_1259/taxi/cache/data.php:1830`
```php
1820:       2 => 'Login',
1821:     ),
1822:     'login_fail' => 
1823:     array (
1824:       1 => 'Неудачная авторизация',
1825:       2 => 'Login fail',
1826:     ),
1827:     'login_success' => 
1828:     array (
1829:       1 => 'Успешная авторизация',
1830:       2 => 'Login success',
1831:     ),
1832:     'logout' => 
1833:     array (
1834:       1 => 'Выход',
1835:       2 => 'Logout',
1836:     ),
1837:     'logout_fail' => 
1838:     array (
1839:       1 => 'Ошибка выхода',
1840:       2 => 'Logout fail',
1841:     ),
1842:     'logout_success' => 
1843:     array (
1844:       1 => 'Успешный выход',
1845:       2 => 'Logout success',
1846:     ),
1847:     'loveseat' => 
1848:     array (
1849:       1 => 'Диван для двоих',
1850:       2 => 'Loveseat',
```

### `archive_17012026_1259/taxi/cache/data.php:2607`
```php
2597:     ),
2598:     'shelf' => 
2599:     array (
2600:       1 => 'Полка',
2601:       2 => 'Shelf',
2602:     ),
2603:     'shipping_application_p' => 
2604:     array (
2605:       1 => 'Оформление заявки на перевозку груза',
2606:     ),
2607:     'signin' => 
2608:     array (
2609:       1 => 'Вход',
2610:       2 => 'Signin',
2611:     ),
2612:     'signup' => 
2613:     array (
2614:       1 => 'Регистрация',
2615:       2 => 'Signup',
2616:     ),
2617:     'sign_in' => 
2618:     array (
2619:       1 => 'Войти',
2620:       2 => 'Sign in',
2621:     ),
2622:     'sign_in_header' => 
2623:     array (
2624:       1 => 'Войти в акаунт',
2625:       2 => 'Signin',
2626:     ),
2627:     'site_name' => 
```

### `archive_17012026_1259/taxi/cache/data.php:2610`
```php
2600:       1 => 'Полка',
2601:       2 => 'Shelf',
2602:     ),
2603:     'shipping_application_p' => 
2604:     array (
2605:       1 => 'Оформление заявки на перевозку груза',
2606:     ),
2607:     'signin' => 
2608:     array (
2609:       1 => 'Вход',
2610:       2 => 'Signin',
2611:     ),
2612:     'signup' => 
2613:     array (
2614:       1 => 'Регистрация',
2615:       2 => 'Signup',
2616:     ),
2617:     'sign_in' => 
2618:     array (
2619:       1 => 'Войти',
2620:       2 => 'Sign in',
2621:     ),
2622:     'sign_in_header' => 
2623:     array (
2624:       1 => 'Войти в акаунт',
2625:       2 => 'Signin',
2626:     ),
2627:     'site_name' => 
2628:     array (
2629:       1 => 'tickets4games',
2630:       2 => 'tickets4games',
```

### `archive_17012026_1259/taxi/cache/data.php:2625`
```php
2615:       2 => 'Signup',
2616:     ),
2617:     'sign_in' => 
2618:     array (
2619:       1 => 'Войти',
2620:       2 => 'Sign in',
2621:     ),
2622:     'sign_in_header' => 
2623:     array (
2624:       1 => 'Войти в акаунт',
2625:       2 => 'Signin',
2626:     ),
2627:     'site_name' => 
2628:     array (
2629:       1 => 'tickets4games',
2630:       2 => 'tickets4games',
2631:     ),
2632:     'small_box' => 
2633:     array (
2634:       1 => 'Малая коробка',
2635:       2 => 'Small box',
2636:     ),
2637:     'small_fridge' => 
2638:     array (
2639:       1 => 'Маленький холодильник',
2640:       2 => 'Small fridge',
2641:     ),
2642:     'sofa' => 
2643:     array (
2644:       1 => 'Оттоманка',
2645:       2 => 'Sofa',
```

### `archive_17012026_1259/taxi/controllers/c_google.php:43`
```php
33: 	if (is_array($id))
34: 	{
35: 		if ($id['error'] == 'user not found')
36: 		{
37: 			redirect("$taxiAppUrl?u_email=".urlencode($email)."&u_name=".urlencode($name)); 
38: 		}
39: 		else
40: 		{
41: 		    if (substr($id['error'],0,6) == 'trying')
42: 			{
43: 				$auth_hash = auth_user($_SESSION[UID]);
44: 				if (empty($auth_hash)) header("Location: "."$taxiAppUrl?error=".urlencode('deleted or banned user'));
45: 				redirect("{$taxiAppUrl}?auth_hash=".urlencode($auth_hash));
46: 			}			
47: 			redirect("$taxiAppUrl?error=".urlencode($id['error']));
48: 		}
49: 	}
50: 	else
51: 	{
52: 		$auth_hash = auth_user($id);
53: 		if (empty($auth_hash)) header("Location: "."$taxiAppUrl?error=".urlencode('deleted or banned user'));
54: 		redirect("{$taxiAppUrl}?auth_hash=".urlencode($auth_hash)); 
55: 	}
56: } else {
57: 	echo "<a href='".$client->createAuthUrl()."'>Google Login</a>";
58: }
59: ?>
```

### `archive_17012026_1259/taxi/controllers/c_google.php:52`
```php
42: 			{
43: 				$auth_hash = auth_user($_SESSION[UID]);
44: 				if (empty($auth_hash)) header("Location: "."$taxiAppUrl?error=".urlencode('deleted or banned user'));
45: 				redirect("{$taxiAppUrl}?auth_hash=".urlencode($auth_hash));
46: 			}			
47: 			redirect("$taxiAppUrl?error=".urlencode($id['error']));
48: 		}
49: 	}
50: 	else
51: 	{
52: 		$auth_hash = auth_user($id);
53: 		if (empty($auth_hash)) header("Location: "."$taxiAppUrl?error=".urlencode('deleted or banned user'));
54: 		redirect("{$taxiAppUrl}?auth_hash=".urlencode($auth_hash)); 
55: 	}
56: } else {
57: 	echo "<a href='".$client->createAuthUrl()."'>Google Login</a>";
58: }
59: ?>
```

### `archive_17012026_1259/taxi/controllers/c_google.php:57`
```php
47: 			redirect("$taxiAppUrl?error=".urlencode($id['error']));
48: 		}
49: 	}
50: 	else
51: 	{
52: 		$auth_hash = auth_user($id);
53: 		if (empty($auth_hash)) header("Location: "."$taxiAppUrl?error=".urlencode('deleted or banned user'));
54: 		redirect("{$taxiAppUrl}?auth_hash=".urlencode($auth_hash)); 
55: 	}
56: } else {
57: 	echo "<a href='".$client->createAuthUrl()."'>Google Login</a>";
58: }
59: ?>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:71`
```php
61: 			<a href="<?php echo url('control_config_role.php'); ?>">Edit config role user</a>
62: 		</div>
63: 		<fieldset class="form"><legend title="Доступна регистрация только Клиента и Водителя. Разрешена для авторизованного пользователя, если это только администратор. Должен быть указан или телефон или емейл (можно и то и другое). При указании емейла пароль приходит на почту. При указании телефона на смс - это пока не реализовано.">Регистрация</legend>
64: 			<form id="register" action="api/v1/register/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">	
65: 				<label><span>Имя</span><input name="u_name" type="text"></label>
66: 				<label><span>Телефон</span><input name="u_phone" type="text"></label>
67: 				<label><span>Емейл</span><input name="u_email" type="text"></label>	
68: 				<label><span>Телеграм идентификатор</span><input name="u_tg" type="text"></label>
69: 				<label><span>Ватсап идентификатор</span><input name="u_wa" type="text"></label>
70: 				<label class="json_key array"><span>архив дополнительных параметров водителя</span><input data-name="u_details" type="text" value="{}"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
71: 				<label class="json_key"><span>пароль</span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
72: 				<label>
73: 					<span>Роль</span>				
74: 					<select name="u_role">
75: 						<option value="1" selected>Клиент</option><option value="2">Водитель</option>
76: 						<option value="3">Диспетчер</option><option value="4">Администратор</option>
77: 						<option value="5">Агент</option>
78: 					</select>
79: 				</label>
80: 				<label><span>Реферальный код</span><input name="ref_code" type="text"></label>
81: 				<label><span>Отдавать токен</span><input class="checkbox" name="st" type="checkbox" value=""></label>
82: 				<input id="register_data" data-name="data" type="hidden">
83: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
84: 			</form>
85: 		</fieldset>
86: 		<fieldset class="form"><legend title="Доступна только для неавторизованного пользователя.">Авторизация</legend>
87: 			<form id="auth" action="api/v1/auth/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
88: 				<label><span>Логин</span><input name="login" type="text"></label>
89: 				<label><span>Пароль</span><input name="password" type="password"></label>
90: 				<label>
91: 					<span>Тип логина</span>				
```

### `archive_17012026_1259/taxi/controllers/c_index.php:88`
```php
78: 					</select>
79: 				</label>
80: 				<label><span>Реферальный код</span><input name="ref_code" type="text"></label>
81: 				<label><span>Отдавать токен</span><input class="checkbox" name="st" type="checkbox" value=""></label>
82: 				<input id="register_data" data-name="data" type="hidden">
83: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
84: 			</form>
85: 		</fieldset>
86: 		<fieldset class="form"><legend title="Доступна только для неавторизованного пользователя.">Авторизация</legend>
87: 			<form id="auth" action="api/v1/auth/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
88: 				<label><span>Логин</span><input name="login" type="text"></label>
89: 				<label><span>Пароль</span><input name="password" type="password"></label>
90: 				<label>
91: 					<span>Тип логина</span>				
92: 					<select  id="auth_type">
93: 
94: 						<option value="e-mail" selected>Авторизация через емейл u_email</option>
95: 						<option value="phone">Авторизация через телефон u_phone</option>
96: 						<option value="tg">Авторизация через идентификатор телеграма u_tg</option>
97: 						<option value="wa">Авторизация через идентификатор ватсапа u_wa</option>
98: 						<option value="wa_tel">Авторизация через телефон ватсапа u_wa_tel</option>
99: 						<option value="tg_tel">Авторизация через телефон телеграма u_tg_tel</option>
100: 						<option value="tg_link">Авторизация через ссылку телеграма u_tg_link</option>
101: 						<option value="phone_wa_id">Авторизация через идентификатор ватсапа телефона u_phone_wa_id</option>
102: 						<option value="phone_tg_id">Авторизация через идентификатор телеграма телефона u_phone_tg_id</option>
103: 						<option value="phone_tg_link">Авторизация через ссылку телеграма телефона u_phone_tg_link</option>
104: 
105: 						<option value="phone_code">Авторизация через код, отправленный смс сообщением на телефон u_phone</option>
106: 						<option value="e-mail_code">Авторизация через код, отправленный на емейл u_email</option>
107: 						<option value="whatsapp">Авторизация через код, отправленный ватцапом на телефон u_phone</option>
108: 						<option value="telegram">Авторизация через код, отправленный телеграмом на телефон u_phone (не поддерживается)</option>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:89`
```php
79: 				</label>
80: 				<label><span>Реферальный код</span><input name="ref_code" type="text"></label>
81: 				<label><span>Отдавать токен</span><input class="checkbox" name="st" type="checkbox" value=""></label>
82: 				<input id="register_data" data-name="data" type="hidden">
83: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
84: 			</form>
85: 		</fieldset>
86: 		<fieldset class="form"><legend title="Доступна только для неавторизованного пользователя.">Авторизация</legend>
87: 			<form id="auth" action="api/v1/auth/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
88: 				<label><span>Логин</span><input name="login" type="text"></label>
89: 				<label><span>Пароль</span><input name="password" type="password"></label>
90: 				<label>
91: 					<span>Тип логина</span>				
92: 					<select  id="auth_type">
93: 
94: 						<option value="e-mail" selected>Авторизация через емейл u_email</option>
95: 						<option value="phone">Авторизация через телефон u_phone</option>
96: 						<option value="tg">Авторизация через идентификатор телеграма u_tg</option>
97: 						<option value="wa">Авторизация через идентификатор ватсапа u_wa</option>
98: 						<option value="wa_tel">Авторизация через телефон ватсапа u_wa_tel</option>
99: 						<option value="tg_tel">Авторизация через телефон телеграма u_tg_tel</option>
100: 						<option value="tg_link">Авторизация через ссылку телеграма u_tg_link</option>
101: 						<option value="phone_wa_id">Авторизация через идентификатор ватсапа телефона u_phone_wa_id</option>
102: 						<option value="phone_tg_id">Авторизация через идентификатор телеграма телефона u_phone_tg_id</option>
103: 						<option value="phone_tg_link">Авторизация через ссылку телеграма телефона u_phone_tg_link</option>
104: 
105: 						<option value="phone_code">Авторизация через код, отправленный смс сообщением на телефон u_phone</option>
106: 						<option value="e-mail_code">Авторизация через код, отправленный на емейл u_email</option>
107: 						<option value="whatsapp">Авторизация через код, отправленный ватцапом на телефон u_phone</option>
108: 						<option value="telegram">Авторизация через код, отправленный телеграмом на телефон u_phone (не поддерживается)</option>
109: 						<option value="whatsapp_id">Авторизация через код, отправленный на ватцапом на идентификатор u_wa (не поддерживается)</option>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:95`
```php
85: 		</fieldset>
86: 		<fieldset class="form"><legend title="Доступна только для неавторизованного пользователя.">Авторизация</legend>
87: 			<form id="auth" action="api/v1/auth/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
88: 				<label><span>Логин</span><input name="login" type="text"></label>
89: 				<label><span>Пароль</span><input name="password" type="password"></label>
90: 				<label>
91: 					<span>Тип логина</span>				
92: 					<select  id="auth_type">
93: 
94: 						<option value="e-mail" selected>Авторизация через емейл u_email</option>
95: 						<option value="phone">Авторизация через телефон u_phone</option>
96: 						<option value="tg">Авторизация через идентификатор телеграма u_tg</option>
97: 						<option value="wa">Авторизация через идентификатор ватсапа u_wa</option>
98: 						<option value="wa_tel">Авторизация через телефон ватсапа u_wa_tel</option>
99: 						<option value="tg_tel">Авторизация через телефон телеграма u_tg_tel</option>
100: 						<option value="tg_link">Авторизация через ссылку телеграма u_tg_link</option>
101: 						<option value="phone_wa_id">Авторизация через идентификатор ватсапа телефона u_phone_wa_id</option>
102: 						<option value="phone_tg_id">Авторизация через идентификатор телеграма телефона u_phone_tg_id</option>
103: 						<option value="phone_tg_link">Авторизация через ссылку телеграма телефона u_phone_tg_link</option>
104: 
105: 						<option value="phone_code">Авторизация через код, отправленный смс сообщением на телефон u_phone</option>
106: 						<option value="e-mail_code">Авторизация через код, отправленный на емейл u_email</option>
107: 						<option value="whatsapp">Авторизация через код, отправленный ватцапом на телефон u_phone</option>
108: 						<option value="telegram">Авторизация через код, отправленный телеграмом на телефон u_phone (не поддерживается)</option>
109: 						<option value="whatsapp_id">Авторизация через код, отправленный на ватцапом на идентификатор u_wa (не поддерживается)</option>
110: 						<option value="telegram_id">Авторизация через код, отправленный телеграмом на идентификатор tg</option>
111: 						<option value="whatsapp_tel">Авторизация через код, отправленный ватцапом на телефон u_wa_tel</option>
112: 						<option value="telegram_tel">Авторизация через код, отправленный телеграмом на телефон u_tg_tel (не поддерживается)</option>
113: 						<option value="telegram_link">Авторизация через код, отправленный телеграмом на ссылку u_tg_link (не поддерживается)</option>
114: 						<option value="whatsapp_ph_id">Авторизация через код, отправленный ватцапом на идентификатор u_phone_wa_id (не поддерживается)</option>
115: 						<option value="telegram_ph_id">Авторизация через код, отправленный телеграмом на идентификатор u_phone_tg_id</option>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:179`
```php
169: 				"password"				пароль
170: 			Ключи массива u_details проверяются в зависимости от константы data.site_constants.u_details_valid_keys.	
171: 			Парсинг u_name происходит по следующей схеме:
172: 				Первая подстрока без пробельных символов считается именем.
173: 				Если подстрок без пробелов больше одной, то последняя рассматривается как фамилия.
174: 				Если подстрок без пробелов больше двух, то текст между первой и последней это отчество.
175: 			Если ref_code не определен, то u_id реферера определяется из куки reco.
176: 			Если кука не определена или имеет неправильно значение, реферер берется из таблицы реферальных ip.
177: 			Если и в таблице ip пользователя не найден, то реферер не указывается.
178: 			При проверке емейла, телефона, телеграм идентификатора на занятость сообщение точно указывает причину:
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:185`
```php
175: 			Если ref_code не определен, то u_id реферера определяется из куки reco.
176: 			Если кука не определена или имеет неправильно значение, реферер берется из таблицы реферальных ip.
177: 			Если и в таблице ip пользователя не найден, то реферер не указывается.
178: 			При проверке емейла, телефона, телеграм идентификатора на занятость сообщение точно указывает причину:
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
```

### `archive_17012026_1259/taxi/controllers/c_index.php:186`
```php
176: 			Если кука не определена или имеет неправильно значение, реферер берется из таблицы реферальных ip.
177: 			Если и в таблице ip пользователя не найден, то реферер не указывается.
178: 			При проверке емейла, телефона, телеграм идентификатора на занятость сообщение точно указывает причину:
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:187`
```php
177: 			Если и в таблице ip пользователя не найден, то реферер не указывается.
178: 			При проверке емейла, телефона, телеграм идентификатора на занятость сообщение точно указывает причину:
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
```

### `archive_17012026_1259/taxi/controllers/c_index.php:188`
```php
178: 			При проверке емейла, телефона, телеграм идентификатора на занятость сообщение точно указывает причину:
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:196`
```php
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:197`
```php
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:198`
```php
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
```

### `archive_17012026_1259/taxi/controllers/c_index.php:200`
```php
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:201`
```php
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
```

### `archive_17012026_1259/taxi/controllers/c_index.php:203`
```php
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:204`
```php
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:207`
```php
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:208`
```php
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
```

### `archive_17012026_1259/taxi/controllers/c_index.php:210`
```php
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
```

### `archive_17012026_1259/taxi/controllers/c_index.php:211`
```php
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:214`
```php
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:215`
```php
205: 				при совпадении происходит авторизация
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:217`
```php
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
```

### `archive_17012026_1259/taxi/controllers/c_index.php:218`
```php
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
```

### `archive_17012026_1259/taxi/controllers/c_index.php:221`
```php
211: 				код, указанный в password, сравнивается с кодом для пользователя емейла, указанного в login,
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
239: 					"u_ban":			{
240: 										"auth":			"число активных банов на авторизацию",
241: 										"order":		"число активных банов на создание или получения поездки",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:222`
```php
212: 				при совпадении происходит авторизация
213: 				
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
239: 					"u_ban":			{
240: 										"auth":			"число активных банов на авторизацию",
241: 										"order":		"число активных банов на создание или получения поездки",
242: 										"blog_topic":	"число активных банов на создание темы в блоге",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:224`
```php
214: 			Если type='phone_code' и password пустой, то
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
239: 					"u_ban":			{
240: 										"auth":			"число активных банов на авторизацию",
241: 										"order":		"число активных банов на создание или получения поездки",
242: 										"blog_topic":	"число активных банов на создание темы в блоге",
243: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
244: 										}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:225`
```php
215: 				на телефон u_phone, указанном в login, отправляется через смс четырехзначный код.
216: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
217: 			Если type='phone_code' и password указан, то	
218: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
219: 				при совпадении происходит авторизация
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
239: 					"u_ban":			{
240: 										"auth":			"число активных банов на авторизацию",
241: 										"order":		"число активных банов на создание или получения поездки",
242: 										"blog_topic":	"число активных банов на создание темы в блоге",
243: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
244: 										}
245: 					"u_active":			"1 или 0",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:230`
```php
220: 
221: 			Если type='telegram_id' и password пустой, то
222: 				на телеграм u_tg, указанном в login, высылается четырехзначный код.
223: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
224: 			Если type='telegram_id' и password указан, то	
225: 				код, указанный в password, сравнивается с кодом для пользователя телеграма, указанного в login,
226: 				при совпадении происходит авторизация
227: 
228: 			Ответ сервера:
229: 			{'code':'200','status':'success',
230: 				"auth_user":{
231: 					"u_id":				"идентификатор пользователя",
232: 					"u_name":			"имя пользователя",
233: 					"u_family":			"фамилия пользователя",
234: 					"u_middle":			"отчество пользователя",
235: 					"u_email":			"емейл пользователя",
236: 					"u_phone":			"телефон пользователя",
237: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
238: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
239: 					"u_ban":			{
240: 										"auth":			"число активных банов на авторизацию",
241: 										"order":		"число активных банов на создание или получения поездки",
242: 										"blog_topic":	"число активных банов на создание темы в блоге",
243: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
244: 										}
245: 					"u_active":			"1 или 0",
246: 					"u_photo":			"ссылка на фото",
247: 					"u_birthday":		"день рождения пользователя в виде год-месяц-день",
248: 					"u_phone_checked":	"0 или 1",
249: 					"u_lang":			"идентификатор языка, выбранного пользователем",	data.langs
250: 					"u_currency":		"iso4217 код валюты, выбранной пользователем",		data.currencies
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4183`
```php
4173: 							"owner":		"владелец контакта",
4174: 							"o_type":		"идентификатор типа владельца контакта", 				data.owner_types
4175: 							"co_class":		"идентификатор типа контактных данных", 				data.contact_classes
4176: 							"cid":			"идентификатор контакта в родной среде",
4177: 							"link":			"уникальное имя ссылка контакта в родной среде",
4178: 							"is_bot":		"0 или 1, бот ли это",
4179: 							"active":		"0 или 1",
4180: 							"number"		"телефон контакта в родной среде",			не админу доступен только для o_type=1
4181: 							"key1"			"ключ1 для доступа к контакту",				не админу доступен только для o_type=1
4182: 							"key2"			"ключ2 для доступа к контакту",				не админу доступен только для o_type=1
4183: 							"token"			"токен для доступа к контакту",				не админу доступен только для o_type=1
4184: 							"hash"			"хеш для доступа к контакту",				не админу доступен только для o_type=1
4185: 							"secret"		"секретное слово для доступа к контакту",	не админу доступен только для o_type=1
4186: 							"host"			"хост для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4187: 							"port"			"порт для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4188: 							"login"			"логин для доступа",						не админу доступен только для o_type=1
4189: 							"password"		"пароль для доступа",						не админу доступен только для o_type=1
4190: 							"smtpsecure"	"протокол шифрования для доступа, 			не админу доступен только для o_type=1
4191: 											как вариант для емейла"
4192: 							"fromname"		"имя отправителя для доступа 				не админу доступен только для o_type=1
4193: 											как вариант для емейла"			
4194: 							<?php
4195: 								$sql_name = $sql_description = array();			
4196: 								foreach (taxi::$data['langs'] as $lang)
4197: 								{
4198: 									$sql_name[] = "\"{$lang['iso']}\"";
4199: 									$sql_description[] = "\"about_{$lang['iso']}\"";
4200: 								}
4201: 								$sql_name = implode(",\n							",$sql_name);
4202: 								$sql_description = implode(",\n							",$sql_description);	
4203: 								echo "$sql_name,\n							$sql_description";
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4188`
```php
4178: 							"is_bot":		"0 или 1, бот ли это",
4179: 							"active":		"0 или 1",
4180: 							"number"		"телефон контакта в родной среде",			не админу доступен только для o_type=1
4181: 							"key1"			"ключ1 для доступа к контакту",				не админу доступен только для o_type=1
4182: 							"key2"			"ключ2 для доступа к контакту",				не админу доступен только для o_type=1
4183: 							"token"			"токен для доступа к контакту",				не админу доступен только для o_type=1
4184: 							"hash"			"хеш для доступа к контакту",				не админу доступен только для o_type=1
4185: 							"secret"		"секретное слово для доступа к контакту",	не админу доступен только для o_type=1
4186: 							"host"			"хост для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4187: 							"port"			"порт для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4188: 							"login"			"логин для доступа",						не админу доступен только для o_type=1
4189: 							"password"		"пароль для доступа",						не админу доступен только для o_type=1
4190: 							"smtpsecure"	"протокол шифрования для доступа, 			не админу доступен только для o_type=1
4191: 											как вариант для емейла"
4192: 							"fromname"		"имя отправителя для доступа 				не админу доступен только для o_type=1
4193: 											как вариант для емейла"			
4194: 							<?php
4195: 								$sql_name = $sql_description = array();			
4196: 								foreach (taxi::$data['langs'] as $lang)
4197: 								{
4198: 									$sql_name[] = "\"{$lang['iso']}\"";
4199: 									$sql_description[] = "\"about_{$lang['iso']}\"";
4200: 								}
4201: 								$sql_name = implode(",\n							",$sql_name);
4202: 								$sql_description = implode(",\n							",$sql_description);	
4203: 								echo "$sql_name,\n							$sql_description";
4204: 							?> 
4205: 						},	
4206: 						...
4207: 					}
4208: 			},
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4189`
```php
4179: 							"active":		"0 или 1",
4180: 							"number"		"телефон контакта в родной среде",			не админу доступен только для o_type=1
4181: 							"key1"			"ключ1 для доступа к контакту",				не админу доступен только для o_type=1
4182: 							"key2"			"ключ2 для доступа к контакту",				не админу доступен только для o_type=1
4183: 							"token"			"токен для доступа к контакту",				не админу доступен только для o_type=1
4184: 							"hash"			"хеш для доступа к контакту",				не админу доступен только для o_type=1
4185: 							"secret"		"секретное слово для доступа к контакту",	не админу доступен только для o_type=1
4186: 							"host"			"хост для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4187: 							"port"			"порт для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4188: 							"login"			"логин для доступа",						не админу доступен только для o_type=1
4189: 							"password"		"пароль для доступа",						не админу доступен только для o_type=1
4190: 							"smtpsecure"	"протокол шифрования для доступа, 			не админу доступен только для o_type=1
4191: 											как вариант для емейла"
4192: 							"fromname"		"имя отправителя для доступа 				не админу доступен только для o_type=1
4193: 											как вариант для емейла"			
4194: 							<?php
4195: 								$sql_name = $sql_description = array();			
4196: 								foreach (taxi::$data['langs'] as $lang)
4197: 								{
4198: 									$sql_name[] = "\"{$lang['iso']}\"";
4199: 									$sql_description[] = "\"about_{$lang['iso']}\"";
4200: 								}
4201: 								$sql_name = implode(",\n							",$sql_name);
4202: 								$sql_description = implode(",\n							",$sql_description);	
4203: 								echo "$sql_name,\n							$sql_description";
4204: 							?> 
4205: 						},	
4206: 						...
4207: 					}
4208: 			},
4209: 			"auth_user":{
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4257`
```php
4247: 					data		JSON.stringify({
4248: 										"owner":		владелец контакта
4249: 										"o_type":		идентификатор типа владельца контакта 		data.owner_types
4250: 										"co_class":		идентификатор типа контактных данных 		data.contact_classes необходимо
4251: 										"number"		телефон контакта в родной среде
4252: 										"cid":			идентификатор контакта в родной среде
4253: 										"link":			уникальное имя ссылка контакта в родной среде
4254: 										"is_bot":		0 или 1, бот ли это
4255: 										"key1"			ключ1 для доступа к контакту
4256: 										"key2"			ключ2 для доступа к контакту
4257: 										"token"			токен для доступа к контакту
4258: 										"hash"			хеш для доступа к контакту
4259: 										"secret"		секретное слово для доступа к контакту
4260: 										"host"			хост для доступа, как вариант для емейла
4261: 										"port"			порт для доступа, как вариант для емейла
4262: 										"login"			логин для доступа
4263: 										"password"		пароль для доступа
4264: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4265: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4266: 										"active":		0 или 1
4267: 									})
4268: 				Админ может создавать любой контакт.
4269: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4270: 				Без указания o_type назначается 1.
4271: 				Если o_type указан и не равен 1, то необходимо указывать owner.
4272: 				Если же o_type не указан или равен 1, то без указания owner=авторизованный_u_id.
4273: 				Ответ сервера:
4274: 				{'code':'200','status':'success'
4275: 					"data":{
4276: 						"affected_fields":		[..],			список валидных ключей
4277: 						"forbidden_fields:		[..],			список невалидных ключей
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4262`
```php
4252: 										"cid":			идентификатор контакта в родной среде
4253: 										"link":			уникальное имя ссылка контакта в родной среде
4254: 										"is_bot":		0 или 1, бот ли это
4255: 										"key1"			ключ1 для доступа к контакту
4256: 										"key2"			ключ2 для доступа к контакту
4257: 										"token"			токен для доступа к контакту
4258: 										"hash"			хеш для доступа к контакту
4259: 										"secret"		секретное слово для доступа к контакту
4260: 										"host"			хост для доступа, как вариант для емейла
4261: 										"port"			порт для доступа, как вариант для емейла
4262: 										"login"			логин для доступа
4263: 										"password"		пароль для доступа
4264: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4265: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4266: 										"active":		0 или 1
4267: 									})
4268: 				Админ может создавать любой контакт.
4269: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4270: 				Без указания o_type назначается 1.
4271: 				Если o_type указан и не равен 1, то необходимо указывать owner.
4272: 				Если же o_type не указан или равен 1, то без указания owner=авторизованный_u_id.
4273: 				Ответ сервера:
4274: 				{'code':'200','status':'success'
4275: 					"data":{
4276: 						"affected_fields":		[..],			список валидных ключей
4277: 						"forbidden_fields:		[..],			список невалидных ключей
4278: 						"co_id":				"идентификатор созданного контакта"
4279: 					}
4280: 				}</pre>
4281: 		<fieldset class="form"><legend title="Создает контакт.">Создание контакта</legend>
4282: 			<form id="create_contact" class="complex" action="api/v1/contact/create/" data-action="api/v1/contact/create/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4263`
```php
4253: 										"link":			уникальное имя ссылка контакта в родной среде
4254: 										"is_bot":		0 или 1, бот ли это
4255: 										"key1"			ключ1 для доступа к контакту
4256: 										"key2"			ключ2 для доступа к контакту
4257: 										"token"			токен для доступа к контакту
4258: 										"hash"			хеш для доступа к контакту
4259: 										"secret"		секретное слово для доступа к контакту
4260: 										"host"			хост для доступа, как вариант для емейла
4261: 										"port"			порт для доступа, как вариант для емейла
4262: 										"login"			логин для доступа
4263: 										"password"		пароль для доступа
4264: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4265: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4266: 										"active":		0 или 1
4267: 									})
4268: 				Админ может создавать любой контакт.
4269: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4270: 				Без указания o_type назначается 1.
4271: 				Если o_type указан и не равен 1, то необходимо указывать owner.
4272: 				Если же o_type не указан или равен 1, то без указания owner=авторизованный_u_id.
4273: 				Ответ сервера:
4274: 				{'code':'200','status':'success'
4275: 					"data":{
4276: 						"affected_fields":		[..],			список валидных ключей
4277: 						"forbidden_fields:		[..],			список невалидных ключей
4278: 						"co_id":				"идентификатор созданного контакта"
4279: 					}
4280: 				}</pre>
4281: 		<fieldset class="form"><legend title="Создает контакт.">Создание контакта</legend>
4282: 			<form id="create_contact" class="complex" action="api/v1/contact/create/" data-action="api/v1/contact/create/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
4283: 				<label class="json_key"><span>владелец контакта</span><input data-name="owner" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>			
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4292`
```php
4282: 			<form id="create_contact" class="complex" action="api/v1/contact/create/" data-action="api/v1/contact/create/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
4283: 				<label class="json_key"><span>владелец контакта</span><input data-name="owner" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>			
4284: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4285: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4286: 				<label class="json_key"><span>телефон контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="number" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4287: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4288: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4289: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4290: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4291: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4292: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4293: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4294: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4295: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4296: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4297: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4298: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4299: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4300: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4301: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4302: 				<?php
4303: 					$sql_name = $sql_description = array();			
4304: 					foreach (taxi::$data['langs'] as $lang)
4305: 					{
4306: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4307: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4308: 					}
4309: 					$sql_name = implode("\n				",$sql_name);
4310: 					$sql_description = implode("\n				",$sql_description);	
4311: 					echo "$sql_name\n				$sql_description";
4312: 				?> 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4297`
```php
4287: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4288: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4289: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4290: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4291: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4292: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4293: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4294: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4295: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4296: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4297: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4298: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4299: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4300: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4301: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4302: 				<?php
4303: 					$sql_name = $sql_description = array();			
4304: 					foreach (taxi::$data['langs'] as $lang)
4305: 					{
4306: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4307: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4308: 					}
4309: 					$sql_name = implode("\n				",$sql_name);
4310: 					$sql_description = implode("\n				",$sql_description);	
4311: 					echo "$sql_name\n				$sql_description";
4312: 				?> 
4313: 				<input id="create_contact_data" name="data" type="hidden">
4314: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
4315: 			</form>
4316: 		</fieldset>
4317: 		<pre>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4298`
```php
4288: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4289: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4290: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4291: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4292: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4293: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4294: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4295: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4296: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4297: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4298: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4299: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4300: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4301: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4302: 				<?php
4303: 					$sql_name = $sql_description = array();			
4304: 					foreach (taxi::$data['langs'] as $lang)
4305: 					{
4306: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4307: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4308: 					}
4309: 					$sql_name = implode("\n				",$sql_name);
4310: 					$sql_description = implode("\n				",$sql_description);	
4311: 					echo "$sql_name\n				$sql_description";
4312: 				?> 
4313: 				<input id="create_contact_data" name="data" type="hidden">
4314: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
4315: 			</form>
4316: 		</fieldset>
4317: 		<pre>
4318: 		https://ibronevik.ru/taxi/api/v1/contact/edit									POST
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4333`
```php
4323: 					data		JSON.stringify({
4324: 										"owner":		владелец контакта													только админ
4325: 										"o_type":		идентификатор типа владельца контакта 		data.owner_types		только админ
4326: 										"co_class":		идентификатор типа контактных данных 		data.contact_classes
4327: 										"number"		телефон контакта в родной среде
4328: 										"cid":			идентификатор контакта в родной среде
4329: 										"link":			уникальное имя ссылка контакта в родной среде
4330: 										"is_bot":		0 или 1, бот ли это
4331: 										"key1"			ключ1 для доступа к контакту
4332: 										"key2"			ключ2 для доступа к контакту
4333: 										"token"			токен для доступа к контакту
4334: 										"hash"			хеш для доступа к контакту
4335: 										"secret"		секретное слово для доступа к контакту
4336: 										"host"			хост для доступа, как вариант для емейла
4337: 										"port"			порт для доступа, как вариант для емейла
4338: 										"login"			логин для доступа
4339: 										"password"		пароль для доступа
4340: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4341: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4342: 										"active":		0 или 1
4343: 									})
4344: 				Админ может редактировать любой контакт.
4345: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4346: 				Ответ сервера:
4347: 				{'code':'200','status':'success'
4348: 					"data":{
4349: 						"affected_fields":		[..],			список валидных ключей
4350: 						"forbidden_fields:		[..],			список невалидных ключей
4351: 					}
4352: 				}</pre>
4353: 		<fieldset class="form"><legend title="Редактирует контакт.">Редактирование контакта</legend>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4338`
```php
4328: 										"cid":			идентификатор контакта в родной среде
4329: 										"link":			уникальное имя ссылка контакта в родной среде
4330: 										"is_bot":		0 или 1, бот ли это
4331: 										"key1"			ключ1 для доступа к контакту
4332: 										"key2"			ключ2 для доступа к контакту
4333: 										"token"			токен для доступа к контакту
4334: 										"hash"			хеш для доступа к контакту
4335: 										"secret"		секретное слово для доступа к контакту
4336: 										"host"			хост для доступа, как вариант для емейла
4337: 										"port"			порт для доступа, как вариант для емейла
4338: 										"login"			логин для доступа
4339: 										"password"		пароль для доступа
4340: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4341: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4342: 										"active":		0 или 1
4343: 									})
4344: 				Админ может редактировать любой контакт.
4345: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4346: 				Ответ сервера:
4347: 				{'code':'200','status':'success'
4348: 					"data":{
4349: 						"affected_fields":		[..],			список валидных ключей
4350: 						"forbidden_fields:		[..],			список невалидных ключей
4351: 					}
4352: 				}</pre>
4353: 		<fieldset class="form"><legend title="Редактирует контакт.">Редактирование контакта</legend>
4354: 			<form id="edit_contact" class="complex" action="api/v1/contact/edit/" data-action="api/v1/contact/edit/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
4355: 				<label><span>идентификатор контакта</span><input data-name="co_id" name="co_id" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
4356: 				<label class="json_key"><span>владелец контакта</span><input data-name="owner" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>			
4357: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4358: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4339`
```php
4329: 										"link":			уникальное имя ссылка контакта в родной среде
4330: 										"is_bot":		0 или 1, бот ли это
4331: 										"key1"			ключ1 для доступа к контакту
4332: 										"key2"			ключ2 для доступа к контакту
4333: 										"token"			токен для доступа к контакту
4334: 										"hash"			хеш для доступа к контакту
4335: 										"secret"		секретное слово для доступа к контакту
4336: 										"host"			хост для доступа, как вариант для емейла
4337: 										"port"			порт для доступа, как вариант для емейла
4338: 										"login"			логин для доступа
4339: 										"password"		пароль для доступа
4340: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4341: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4342: 										"active":		0 или 1
4343: 									})
4344: 				Админ может редактировать любой контакт.
4345: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4346: 				Ответ сервера:
4347: 				{'code':'200','status':'success'
4348: 					"data":{
4349: 						"affected_fields":		[..],			список валидных ключей
4350: 						"forbidden_fields:		[..],			список невалидных ключей
4351: 					}
4352: 				}</pre>
4353: 		<fieldset class="form"><legend title="Редактирует контакт.">Редактирование контакта</legend>
4354: 			<form id="edit_contact" class="complex" action="api/v1/contact/edit/" data-action="api/v1/contact/edit/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
4355: 				<label><span>идентификатор контакта</span><input data-name="co_id" name="co_id" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
4356: 				<label class="json_key"><span>владелец контакта</span><input data-name="owner" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>			
4357: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4358: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4359: 				<label class="json_key"><span>телефон контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="number" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4365`
```php
4355: 				<label><span>идентификатор контакта</span><input data-name="co_id" name="co_id" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
4356: 				<label class="json_key"><span>владелец контакта</span><input data-name="owner" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>			
4357: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4358: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4359: 				<label class="json_key"><span>телефон контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="number" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4360: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4361: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4362: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4363: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4364: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4365: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4366: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4367: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4368: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4369: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4370: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4371: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4372: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4373: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4374: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4375: 				<?php
4376: 					$sql_name = $sql_description = array();			
4377: 					foreach (taxi::$data['langs'] as $lang)
4378: 					{
4379: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4380: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4381: 					}
4382: 					$sql_name = implode("\n				",$sql_name);
4383: 					$sql_description = implode("\n				",$sql_description);	
4384: 					echo "$sql_name\n				$sql_description";
4385: 				?> 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4370`
```php
4360: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4361: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4362: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4363: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4364: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4365: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4366: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4367: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4368: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4369: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4370: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4371: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4372: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4373: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4374: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4375: 				<?php
4376: 					$sql_name = $sql_description = array();			
4377: 					foreach (taxi::$data['langs'] as $lang)
4378: 					{
4379: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4380: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4381: 					}
4382: 					$sql_name = implode("\n				",$sql_name);
4383: 					$sql_description = implode("\n				",$sql_description);	
4384: 					echo "$sql_name\n				$sql_description";
4385: 				?> 
4386: 				<input id="edit_contact_data" name="data" type="hidden">
4387: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
4388: 			</form>
4389: 		</fieldset>
4390: 		<pre>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4371`
```php
4361: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4362: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4363: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4364: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4365: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4366: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4367: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4368: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4369: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4370: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4371: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4372: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4373: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4374: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4375: 				<?php
4376: 					$sql_name = $sql_description = array();			
4377: 					foreach (taxi::$data['langs'] as $lang)
4378: 					{
4379: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4380: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4381: 					}
4382: 					$sql_name = implode("\n				",$sql_name);
4383: 					$sql_description = implode("\n				",$sql_description);	
4384: 					echo "$sql_name\n				$sql_description";
4385: 				?> 
4386: 				<input id="edit_contact_data" name="data" type="hidden">
4387: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
4388: 			</form>
4389: 		</fieldset>
4390: 		<pre>
4391: 		https://ibronevik.ru/taxi/api/v1/contact/message/get									POST	
```

### `archive_17012026_1259/taxi/models/api.php:318`
```php
308: 				FROM `ip_referral`
309: 				WHERE 
310: 					`ip` = INET_ATON('" . $ip . "')
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
```

### `archive_17012026_1259/taxi/models/api.php:319`
```php
309: 				WHERE 
310: 					`ip` = INET_ATON('" . $ip . "')
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
```

### `archive_17012026_1259/taxi/models/api.php:321`
```php
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
```

### `archive_17012026_1259/taxi/models/api.php:325`
```php
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
```

### `archive_17012026_1259/taxi/models/api.php:326`
```php
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
```

### `archive_17012026_1259/taxi/models/api.php:327`
```php
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
```

### `archive_17012026_1259/taxi/models/api.php:338`
```php
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:341`
```php
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
```

### `archive_17012026_1259/taxi/models/api.php:344`
```php
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
```

### `archive_17012026_1259/taxi/models/api.php:346`
```php
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
```

### `archive_17012026_1259/taxi/models/api.php:350`
```php
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
```

### `archive_17012026_1259/taxi/models/api.php:352`
```php
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
372: 						`tips`,
```

### `archive_17012026_1259/taxi/models/api.php:355`
```php
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
350: 				if (empty($password))
351: 				{
352: 					return $this->showError('404', 'error', 'empty password');
353: 				}
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
372: 						`tips`,
373: 						`language_skills`,
374: 						`description`,
375: 						`id_navigation`,
```

### `archive_17012026_1259/taxi/models/api.php:364`
```php
354: 
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
372: 						`tips`,
373: 						`language_skills`,
374: 						`description`,
375: 						`id_navigation`,
376: 						`id_verification_status`,
377: 						`active`,
378: 						`deleted`,
379: 						`birthday_date`
380: 					FROM `users`
381: 					WHERE 
382: 						$sql
383: 					LIMIT 1
384: 					";
```

### `archive_17012026_1259/taxi/models/api.php:365`
```php
355: 				$sql = $this->getLoginSql($type,$login);
356: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
357: 
358: 				$s = "SELECT 
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
372: 						`tips`,
373: 						`language_skills`,
374: 						`description`,
375: 						`id_navigation`,
376: 						`id_verification_status`,
377: 						`active`,
378: 						`deleted`,
379: 						`birthday_date`
380: 					FROM `users`
381: 					WHERE 
382: 						$sql
383: 					LIMIT 1
384: 					";
385: 
```

### `archive_17012026_1259/taxi/models/api.php:392`
```php
382: 						$sql
383: 					LIMIT 1
384: 					";
385: 
386: 				$q = query($s);
387: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
388: 				$d = fetch_assoc($q);
389: 
390: 				if (empty($d['id_user']))
391: 				{
392: 					return $this->showError('404', 'error', 'wrong login');
393: 				}
394: 				elseif (md5(md5($password)) != $d['pwd'])
395: 				{
396: 					return $this->showError('404', 'error', 'wrong password');
397: 				}
398: 				elseif (!empty($d['deleted']))
399: 				{
400: 					return $this->showError('404', 'error', 'deleted user');
401: 				}
402: 				$user_ban_status = get_user_ban_status($d['id_user']);
403: 				if (!empty($user_ban_status['auth']))
404: 				{
405: 					return $this->showError('404', 'error', 'banned user');
406: 				}
407: 			}
408: 			else
409: 			{
410: 				$type_arr = explode(',',$type);
411: 				$type_code = $type_arr[0];
412: 				$send_par = $this->getSendParForAuthCode($type_code);
```

### `archive_17012026_1259/taxi/models/api.php:394`
```php
384: 					";
385: 
386: 				$q = query($s);
387: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
388: 				$d = fetch_assoc($q);
389: 
390: 				if (empty($d['id_user']))
391: 				{
392: 					return $this->showError('404', 'error', 'wrong login');
393: 				}
394: 				elseif (md5(md5($password)) != $d['pwd'])
395: 				{
396: 					return $this->showError('404', 'error', 'wrong password');
397: 				}
398: 				elseif (!empty($d['deleted']))
399: 				{
400: 					return $this->showError('404', 'error', 'deleted user');
401: 				}
402: 				$user_ban_status = get_user_ban_status($d['id_user']);
403: 				if (!empty($user_ban_status['auth']))
404: 				{
405: 					return $this->showError('404', 'error', 'banned user');
406: 				}
407: 			}
408: 			else
409: 			{
410: 				$type_arr = explode(',',$type);
411: 				$type_code = $type_arr[0];
412: 				$send_par = $this->getSendParForAuthCode($type_code);
413: 				if (!empty($send_par['error'])) return $this->showError('404', 'error', $send_par['error']);
414: 				if (empty($type_arr[1]))
```

### `archive_17012026_1259/taxi/models/api.php:396`
```php
386: 				$q = query($s);
387: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
388: 				$d = fetch_assoc($q);
389: 
390: 				if (empty($d['id_user']))
391: 				{
392: 					return $this->showError('404', 'error', 'wrong login');
393: 				}
394: 				elseif (md5(md5($password)) != $d['pwd'])
395: 				{
396: 					return $this->showError('404', 'error', 'wrong password');
397: 				}
398: 				elseif (!empty($d['deleted']))
399: 				{
400: 					return $this->showError('404', 'error', 'deleted user');
401: 				}
402: 				$user_ban_status = get_user_ban_status($d['id_user']);
403: 				if (!empty($user_ban_status['auth']))
404: 				{
405: 					return $this->showError('404', 'error', 'banned user');
406: 				}
407: 			}
408: 			else
409: 			{
410: 				$type_arr = explode(',',$type);
411: 				$type_code = $type_arr[0];
412: 				$send_par = $this->getSendParForAuthCode($type_code);
413: 				if (!empty($send_par['error'])) return $this->showError('404', 'error', $send_par['error']);
414: 				if (empty($type_arr[1]))
415: 				{
416: 					$type = $send_par['type'];
```

### `archive_17012026_1259/taxi/models/api.php:428`
```php
418: 				}
419: 				elseif (!in_array($type_arr[1],$this->auth_type_arr)) 
420: 				{
421: 					return $this->showError('404', 'error', 'wrong auth type');
422: 				}
423: 				else
424: 				{
425: 					$type = $type_arr[1];
426: 				}
427: 
428: 				$sql = $this->getLoginSql($type,$login);
429: 				$sql_login = $this->getLoginField($send_par['type']);
430: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
431: 
432: 				$s = "SELECT 
433: 						`id_user`,
434: 						`deleted`,
435: 						`$sql_login`
436: 					FROM `users`
437: 					WHERE 
438: 						$sql
439: 					LIMIT 1
440: 					";
441: 
442: 				$q = query($s);
443: 				if ($q === false) return $this->showError('404', 'error', 'select of database failed');
444: 				$d = fetch_assoc($q);
445: 
446: 				if (empty($d['id_user']))
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
```

### `archive_17012026_1259/taxi/models/api.php:429`
```php
419: 				elseif (!in_array($type_arr[1],$this->auth_type_arr)) 
420: 				{
421: 					return $this->showError('404', 'error', 'wrong auth type');
422: 				}
423: 				else
424: 				{
425: 					$type = $type_arr[1];
426: 				}
427: 
428: 				$sql = $this->getLoginSql($type,$login);
429: 				$sql_login = $this->getLoginField($send_par['type']);
430: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
431: 
432: 				$s = "SELECT 
433: 						`id_user`,
434: 						`deleted`,
435: 						`$sql_login`
436: 					FROM `users`
437: 					WHERE 
438: 						$sql
439: 					LIMIT 1
440: 					";
441: 
442: 				$q = query($s);
443: 				if ($q === false) return $this->showError('404', 'error', 'select of database failed');
444: 				$d = fetch_assoc($q);
445: 
446: 				if (empty($d['id_user']))
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
449: 				}
```

### `archive_17012026_1259/taxi/models/api.php:435`
```php
425: 					$type = $type_arr[1];
426: 				}
427: 
428: 				$sql = $this->getLoginSql($type,$login);
429: 				$sql_login = $this->getLoginField($send_par['type']);
430: 				if (!empty($sql['error'])) return $this->showError('404', 'error', $sql['error']);
431: 
432: 				$s = "SELECT 
433: 						`id_user`,
434: 						`deleted`,
435: 						`$sql_login`
436: 					FROM `users`
437: 					WHERE 
438: 						$sql
439: 					LIMIT 1
440: 					";
441: 
442: 				$q = query($s);
443: 				if ($q === false) return $this->showError('404', 'error', 'select of database failed');
444: 				$d = fetch_assoc($q);
445: 
446: 				if (empty($d['id_user']))
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
```

### `archive_17012026_1259/taxi/models/api.php:448`
```php
438: 						$sql
439: 					LIMIT 1
440: 					";
441: 
442: 				$q = query($s);
443: 				if ($q === false) return $this->showError('404', 'error', 'select of database failed');
444: 				$d = fetch_assoc($q);
445: 
446: 				if (empty($d['id_user']))
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
```

### `archive_17012026_1259/taxi/models/api.php:452`
```php
442: 				$q = query($s);
443: 				if ($q === false) return $this->showError('404', 'error', 'select of database failed');
444: 				$d = fetch_assoc($q);
445: 
446: 				if (empty($d['id_user']))
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
```

### `archive_17012026_1259/taxi/models/api.php:457`
```php
447: 				{
448: 					return $this->showError('404', 'error', 'wrong phone');
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
```

### `archive_17012026_1259/taxi/models/api.php:459`
```php
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
```

### `archive_17012026_1259/taxi/models/api.php:461`
```php
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
455: 				if (!empty($user_ban_status['auth']))
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
```

### `archive_17012026_1259/taxi/models/api.php:468`
```php
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
```

### `archive_17012026_1259/taxi/models/api.php:470`
```php
460: 
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
```

### `archive_17012026_1259/taxi/models/api.php:471`
```php
461: 				if (empty($password))
462: 				{
463: 					$code = generate_code(4);
464: 					$json_str = '{}';
465: 					switch ($send_par['way'])
466: 					{
467: 						case 'whatsapp':
468: 							switch ($send_par['login'])
469: 							{
470: 								case 'phone':
471: 									if (!send_msg_to_whatsapp($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
472: 									break;
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
```

### `archive_17012026_1259/taxi/models/api.php:487`
```php
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
```

### `archive_17012026_1259/taxi/models/api.php:489`
```php
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
```

### `archive_17012026_1259/taxi/models/api.php:490`
```php
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
```

### `archive_17012026_1259/taxi/models/api.php:491`
```php
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
```

### `archive_17012026_1259/taxi/models/api.php:492`
```php
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
```

### `archive_17012026_1259/taxi/models/api.php:495`
```php
485: 							$body = lang('email_code_body','Code: {$code}',2);
486: 							list($subject,$body) = fill_in_template(array($subject,$body),$dataForEmail);
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
513: 							`code` = '" . $code . "',
514: 							`expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "',
515: 							`auth_type` = '$type_code,$type',
```

### `archive_17012026_1259/taxi/models/api.php:497`
```php
487: 							if (!send_mail($send_par['login_val'],$subject,$body)) return $this->showError('404', 'error', 'sending error');
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
513: 							`code` = '" . $code . "',
514: 							`expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "',
515: 							`auth_type` = '$type_code,$type',
516: 							`json` = '$json_str'
517: 						ON DUPLICATE KEY UPDATE 
```

### `archive_17012026_1259/taxi/models/api.php:498`
```php
488: 							break;
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
513: 							`code` = '" . $code . "',
514: 							`expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "',
515: 							`auth_type` = '$type_code,$type',
516: 							`json` = '$json_str'
517: 						ON DUPLICATE KEY UPDATE 
518: 							`code` = '" . $code . "', `expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "', `json` = '$json_str'
```

### `archive_17012026_1259/taxi/models/api.php:501`
```php
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
497: 								case 'phone':
498: 									return $this->showError('404', 'error', 'telegram phone not supported');
499: 									break;
500: 								case 'id':
501: 									if (!send_msg_to_telegram_id($send_par['login_val'],$code)) return $this->showError('404', 'error', 'sending error');
502: 									break;
503: 								case 'link':
504: 									return $this->showError('404', 'error', 'telegram link not supported');
505: 									break;
506: 							}
507: 							break;
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
513: 							`code` = '" . $code . "',
514: 							`expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "',
515: 							`auth_type` = '$type_code,$type',
516: 							`json` = '$json_str'
517: 						ON DUPLICATE KEY UPDATE 
518: 							`code` = '" . $code . "', `expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "', `json` = '$json_str'
519: 						";
520: 
521: 					$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:1353`
```php
1343: 				{
1344: 					unset($affected_keys['u_details']);
1345: 				}
1346: 			}
1347: 
1348: 			$logins = array();
1349: 			$login_flags = array();
1350: 			if (!empty($affected_keys['u_phone'])) 
1351: 			{
1352: 				if ($data['u_phone'] !== NULL) @$data['u_phone'] = preg_replace('/[^0-9]+/','',$data['u_phone']);
1353: 				$logins[] = 'phone';
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
```

### `archive_17012026_1259/taxi/models/api.php:1354`
```php
1344: 					unset($affected_keys['u_details']);
1345: 				}
1346: 			}
1347: 
1348: 			$logins = array();
1349: 			$login_flags = array();
1350: 			if (!empty($affected_keys['u_phone'])) 
1351: 			{
1352: 				if ($data['u_phone'] !== NULL) @$data['u_phone'] = preg_replace('/[^0-9]+/','',$data['u_phone']);
1353: 				$logins[] = 'phone';
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
```

### `archive_17012026_1259/taxi/models/api.php:1358`
```php
1348: 			$logins = array();
1349: 			$login_flags = array();
1350: 			if (!empty($affected_keys['u_phone'])) 
1351: 			{
1352: 				if ($data['u_phone'] !== NULL) @$data['u_phone'] = preg_replace('/[^0-9]+/','',$data['u_phone']);
1353: 				$logins[] = 'phone';
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
```

### `archive_17012026_1259/taxi/models/api.php:1359`
```php
1349: 			$login_flags = array();
1350: 			if (!empty($affected_keys['u_phone'])) 
1351: 			{
1352: 				if ($data['u_phone'] !== NULL) @$data['u_phone'] = preg_replace('/[^0-9]+/','',$data['u_phone']);
1353: 				$logins[] = 'phone';
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
```

### `archive_17012026_1259/taxi/models/api.php:1363`
```php
1353: 				$logins[] = 'phone';
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
```

### `archive_17012026_1259/taxi/models/api.php:1364`
```php
1354: 				$login_flags[] = 'pflag';
1355: 			}
1356: 			if (!empty($affected_keys['u_email'])) 
1357: 			{
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
```

### `archive_17012026_1259/taxi/models/api.php:1368`
```php
1358: 				$logins[] = 'email';
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
```

### `archive_17012026_1259/taxi/models/api.php:1369`
```php
1359: 				$login_flags[] = 'eflag';
1360: 			}
1361: 			if (!empty($affected_keys['u_tg'])) 
1362: 			{
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
```

### `archive_17012026_1259/taxi/models/api.php:1373`
```php
1363: 				$logins[] = 'tg';
1364: 				$login_flags[] = 'tflag';
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
```

### `archive_17012026_1259/taxi/models/api.php:1375`
```php
1365: 			}
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
```

### `archive_17012026_1259/taxi/models/api.php:1376`
```php
1366: 			if (!empty($affected_keys['u_wa'])) 
1367: 			{
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
```

### `archive_17012026_1259/taxi/models/api.php:1378`
```php
1368: 				$logins[] = 'wa';
1369: 				$login_flags[] = 'wflag';
1370: 			}
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
```

### `archive_17012026_1259/taxi/models/api.php:1381`
```php
1371: 			$sql_field = array();
1372: 			$sql_where = array();
1373: 			foreach($logins as $i=>$login)
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
```

### `archive_17012026_1259/taxi/models/api.php:1385`
```php
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
```

### `archive_17012026_1259/taxi/models/api.php:1387`
```php
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
```

### `archive_17012026_1259/taxi/models/api.php:1389`
```php
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:1390`
```php
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
```

### `archive_17012026_1259/taxi/models/api.php:1393`
```php
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
```

### `archive_17012026_1259/taxi/models/api.php:1397`
```php
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
1414: 
1415: 				$q = query($s);
1416: 				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
1417: 				$d = fetch_assoc($q);
```

### `archive_17012026_1259/taxi/models/api.php:1401`
```php
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
1414: 
1415: 				$q = query($s);
1416: 				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
1417: 				$d = fetch_assoc($q);
1418: 				$msg = array();
1419: 				foreach($login_flags as $i=>$login_flag)
1420: 				{
1421: 					if (!empty($d[$login_flag]))
```

### `archive_17012026_1259/taxi/models/api.php:1403`
```php
1393: 					$$login = $data[$u_login];
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
1414: 
1415: 				$q = query($s);
1416: 				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
1417: 				$d = fetch_assoc($q);
1418: 				$msg = array();
1419: 				foreach($login_flags as $i=>$login_flag)
1420: 				{
1421: 					if (!empty($d[$login_flag]))
1422: 					{
1423: 						$msg[] = "double $logins[$i]";
```

### `archive_17012026_1259/taxi/models/api.php:9060`
```php
9050: 
9051: 			$q = query("COMMIT");
9052: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
9053: 
9054: 			return array(
9055: 				'code' 		=>	'200',
9056: 				'status' 	=>	'success'
9057: 			);
9058: 		}
9059: 
9060: 		public function remind($email = "", $phone = "", $tg = "", $wa = "")
9061: 		{
9062: 		
9063: 			$phone = preg_replace('/[^0-9]+/','',$phone);
9064: 			while (true){
9065: 				$u_data = array('email','phone','tg','wa');
9066: 				foreach($u_data as $u_str)
9067: 				{
9068: 					if (empty($$u_str)) continue;
9069: 					$sql = "`$u_str` = '" . $$u_str . "'";
9070: 					break 2;
9071: 				}
9072: 				return array('error' => 'empty user login');
9073: 			}
9074: 
9075: 			$s = "SELECT 
9076: 					`id_user`,
9077: 					`deleted`,
9078: 					`name`,
9079: 					`family`,
9080: 					`middle`,
```

### `archive_17012026_1259/taxi/models/api.php:9063`
```php
9053: 
9054: 			return array(
9055: 				'code' 		=>	'200',
9056: 				'status' 	=>	'success'
9057: 			);
9058: 		}
9059: 
9060: 		public function remind($email = "", $phone = "", $tg = "", $wa = "")
9061: 		{
9062: 		
9063: 			$phone = preg_replace('/[^0-9]+/','',$phone);
9064: 			while (true){
9065: 				$u_data = array('email','phone','tg','wa');
9066: 				foreach($u_data as $u_str)
9067: 				{
9068: 					if (empty($$u_str)) continue;
9069: 					$sql = "`$u_str` = '" . $$u_str . "'";
9070: 					break 2;
9071: 				}
9072: 				return array('error' => 'empty user login');
9073: 			}
9074: 
9075: 			$s = "SELECT 
9076: 					`id_user`,
9077: 					`deleted`,
9078: 					`name`,
9079: 					`family`,
9080: 					`middle`,
9081: 					`email`
9082: 				FROM `users`
9083: 				WHERE 
```

### `archive_17012026_1259/taxi/models/api.php:9065`
```php
9055: 				'code' 		=>	'200',
9056: 				'status' 	=>	'success'
9057: 			);
9058: 		}
9059: 
9060: 		public function remind($email = "", $phone = "", $tg = "", $wa = "")
9061: 		{
9062: 		
9063: 			$phone = preg_replace('/[^0-9]+/','',$phone);
9064: 			while (true){
9065: 				$u_data = array('email','phone','tg','wa');
9066: 				foreach($u_data as $u_str)
9067: 				{
9068: 					if (empty($$u_str)) continue;
9069: 					$sql = "`$u_str` = '" . $$u_str . "'";
9070: 					break 2;
9071: 				}
9072: 				return array('error' => 'empty user login');
9073: 			}
9074: 
9075: 			$s = "SELECT 
9076: 					`id_user`,
9077: 					`deleted`,
9078: 					`name`,
9079: 					`family`,
9080: 					`middle`,
9081: 					`email`
9082: 				FROM `users`
9083: 				WHERE 
9084: 					$sql
9085: 				LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:9072`
```php
9062: 		
9063: 			$phone = preg_replace('/[^0-9]+/','',$phone);
9064: 			while (true){
9065: 				$u_data = array('email','phone','tg','wa');
9066: 				foreach($u_data as $u_str)
9067: 				{
9068: 					if (empty($$u_str)) continue;
9069: 					$sql = "`$u_str` = '" . $$u_str . "'";
9070: 					break 2;
9071: 				}
9072: 				return array('error' => 'empty user login');
9073: 			}
9074: 
9075: 			$s = "SELECT 
9076: 					`id_user`,
9077: 					`deleted`,
9078: 					`name`,
9079: 					`family`,
9080: 					`middle`,
9081: 					`email`
9082: 				FROM `users`
9083: 				WHERE 
9084: 					$sql
9085: 				LIMIT 1
9086: 				";
9087: 
9088: 			$q = query($s);
9089: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
9090: 			$d = fetch_assoc($q);
9091: 
9092: 			if (empty($d['id_user']))
```

### `archive_17012026_1259/taxi/models/api.php:9094`
```php
9084: 					$sql
9085: 				LIMIT 1
9086: 				";
9087: 
9088: 			$q = query($s);
9089: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
9090: 			$d = fetch_assoc($q);
9091: 
9092: 			if (empty($d['id_user']))
9093: 			{
9094: 				return $this->showError('404', 'error', 'wrong login');
9095: 			}
9096: 			elseif (!empty($d['deleted']))
9097: 			{
9098: 				return $this->showError('404', 'error', 'deleted user');
9099: 			}
9100: 			$user_ban_status = get_user_ban_status($d['id_user']);
9101: 			if (!empty($user_ban_status['auth']))
9102: 			{
9103: 				return $this->showError('404', 'error', 'banned user');
9104: 			}
9105: 
9106: 			$pwd = generate_password(10);
9107: 
9108: 			$s = "UPDATE `users`
9109: 				SET 
9110: 					`pwd` = '" . md5(md5($pwd)) . "'
9111: 				WHERE
9112: 					`id_user` = '" . $d['id_user'] . "'
9113: 				";
9114: 
```

### `archive_17012026_1259/taxi/models/api.php:14403`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:14404`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:14411`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:14412`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20857`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20862`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20863`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20869`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20874`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20875`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21163`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21164`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21183`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21184`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21187`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21188`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21367`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21368`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21387`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21388`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21391`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21392`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:26893`
```php
26883: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
26884: 
26885: 			if (!empty($warning)) $out['warning'] = $warning;
26886: 			return array(
26887: 				'code' 		=>	'200',
26888: 				'status' 	=>	'success',		
26889: 				'data' 		=>	isset($out) ? $out : array()
26890: 			);
26891: 		}
26892: 
26893: 		private $auth_type_arr = array('e-mail','phone','tg','wa','wa_tel','tg_tel','tg_link','phone_wa_id','phone_tg_id','phone_tg_link');
26894: 		private $auth_type_code_arr = array('phone_code','e-mail_code','whatsapp','telegram','whatsapp_id','telegram_id','whatsapp_tel','telegram_tel','telegram_link','whatsapp_ph_id','telegram_ph_id','telegram_ph_link','phone_code_whatsapp','phone_code_telegram');		
26895: 		
26896: 		/*
26897: 			$type_code,$type
26898: 		*/
26899: 		
26900: 		private function getLoginSql($type = '', $login = '')
26901: 		{
26902: 			switch ($type){
26903: 				case 'e-mail':
26904: 					$sql = "`email` = '$login'";
26905: 					break;
26906: 				case 'phone':
26907: 					$sql = "`phone` = '" . preparePhone($login) . "'";
26908: 					break;
26909: 				case 'tg':
26910: 					$sql = "`tg` = '$login'";
26911: 					break;
26912: 				case 'wa':
26913: 					$sql = "`wa` = '$login'";
```

### `archive_17012026_1259/taxi/models/api.php:26894`
```php
26884: 
26885: 			if (!empty($warning)) $out['warning'] = $warning;
26886: 			return array(
26887: 				'code' 		=>	'200',
26888: 				'status' 	=>	'success',		
26889: 				'data' 		=>	isset($out) ? $out : array()
26890: 			);
26891: 		}
26892: 
26893: 		private $auth_type_arr = array('e-mail','phone','tg','wa','wa_tel','tg_tel','tg_link','phone_wa_id','phone_tg_id','phone_tg_link');
26894: 		private $auth_type_code_arr = array('phone_code','e-mail_code','whatsapp','telegram','whatsapp_id','telegram_id','whatsapp_tel','telegram_tel','telegram_link','whatsapp_ph_id','telegram_ph_id','telegram_ph_link','phone_code_whatsapp','phone_code_telegram');		
26895: 		
26896: 		/*
26897: 			$type_code,$type
26898: 		*/
26899: 		
26900: 		private function getLoginSql($type = '', $login = '')
26901: 		{
26902: 			switch ($type){
26903: 				case 'e-mail':
26904: 					$sql = "`email` = '$login'";
26905: 					break;
26906: 				case 'phone':
26907: 					$sql = "`phone` = '" . preparePhone($login) . "'";
26908: 					break;
26909: 				case 'tg':
26910: 					$sql = "`tg` = '$login'";
26911: 					break;
26912: 				case 'wa':
26913: 					$sql = "`wa` = '$login'";
26914: 					break;						
```

### `archive_17012026_1259/taxi/models/api.php:26900`
```php
26890: 			);
26891: 		}
26892: 
26893: 		private $auth_type_arr = array('e-mail','phone','tg','wa','wa_tel','tg_tel','tg_link','phone_wa_id','phone_tg_id','phone_tg_link');
26894: 		private $auth_type_code_arr = array('phone_code','e-mail_code','whatsapp','telegram','whatsapp_id','telegram_id','whatsapp_tel','telegram_tel','telegram_link','whatsapp_ph_id','telegram_ph_id','telegram_ph_link','phone_code_whatsapp','phone_code_telegram');		
26895: 		
26896: 		/*
26897: 			$type_code,$type
26898: 		*/
26899: 		
26900: 		private function getLoginSql($type = '', $login = '')
26901: 		{
26902: 			switch ($type){
26903: 				case 'e-mail':
26904: 					$sql = "`email` = '$login'";
26905: 					break;
26906: 				case 'phone':
26907: 					$sql = "`phone` = '" . preparePhone($login) . "'";
26908: 					break;
26909: 				case 'tg':
26910: 					$sql = "`tg` = '$login'";
26911: 					break;
26912: 				case 'wa':
26913: 					$sql = "`wa` = '$login'";
26914: 					break;						
26915: 				case 'wa_tel':
26916: 					$sql = "`wa_number` = '" . preparePhone($login) . "'";
26917: 					break;						
26918: 				case 'tg_tel':
26919: 					$sql = "`tg_number` = '" . preparePhone($login) . "'";
26920: 					break;
```

### `archive_17012026_1259/taxi/models/api.php:26904`
```php
26894: 		private $auth_type_code_arr = array('phone_code','e-mail_code','whatsapp','telegram','whatsapp_id','telegram_id','whatsapp_tel','telegram_tel','telegram_link','whatsapp_ph_id','telegram_ph_id','telegram_ph_link','phone_code_whatsapp','phone_code_telegram');		
26895: 		
26896: 		/*
26897: 			$type_code,$type
26898: 		*/
26899: 		
26900: 		private function getLoginSql($type = '', $login = '')
26901: 		{
26902: 			switch ($type){
26903: 				case 'e-mail':
26904: 					$sql = "`email` = '$login'";
26905: 					break;
26906: 				case 'phone':
26907: 					$sql = "`phone` = '" . preparePhone($login) . "'";
26908: 					break;
26909: 				case 'tg':
26910: 					$sql = "`tg` = '$login'";
26911: 					break;
26912: 				case 'wa':
26913: 					$sql = "`wa` = '$login'";
26914: 					break;						
26915: 				case 'wa_tel':
26916: 					$sql = "`wa_number` = '" . preparePhone($login) . "'";
26917: 					break;						
26918: 				case 'tg_tel':
26919: 					$sql = "`tg_number` = '" . preparePhone($login) . "'";
26920: 					break;
26921: 				case 'tg_link':
26922: 					$sql = "`tg_link` = '$login'";
26923: 					break;						
26924: 				case 'phone_wa_id':
```

### `archive_17012026_1259/taxi/models/api.php:26906`
```php
26896: 		/*
26897: 			$type_code,$type
26898: 		*/
26899: 		
26900: 		private function getLoginSql($type = '', $login = '')
26901: 		{
26902: 			switch ($type){
26903: 				case 'e-mail':
26904: 					$sql = "`email` = '$login'";
26905: 					break;
26906: 				case 'phone':
26907: 					$sql = "`phone` = '" . preparePhone($login) . "'";
26908: 					break;
26909: 				case 'tg':
26910: 					$sql = "`tg` = '$login'";
26911: 					break;
26912: 				case 'wa':
26913: 					$sql = "`wa` = '$login'";
26914: 					break;						
26915: 				case 'wa_tel':
26916: 					$sql = "`wa_number` = '" . preparePhone($login) . "'";
26917: 					break;						
26918: 				case 'tg_tel':
26919: 					$sql = "`tg_number` = '" . preparePhone($login) . "'";
26920: 					break;
26921: 				case 'tg_link':
26922: 					$sql = "`tg_link` = '$login'";
26923: 					break;						
26924: 				case 'phone_wa_id':
26925: 					$sql = "`phone_wa_id` = '$login'";
26926: 					break;	
```

## 4. Important evidence boundary

В доступном Core Backend archive authentication is centered around `check_auth_user()` and related request/session handling. Это подтверждает authenticated request gate, но login endpoint нельзя считать установленным только по наличию `token`, `u_hash` или `password` в разных PHP contexts.

Для `CONFIRMED` cross-layer relation нужен один и тот же endpoint/value-flow:

```text
Frontend auth call
      ↓
exact backend endpoint
      ↓
response credential / identity
      ↓
frontend state
      ↓
protected request
      ↓
check_auth_user()
```

## 5. Current conclusion

RP-35 v0.2 теперь использует оба source corpus, а не пытается искать backend login handler внутри frontend snapshot.

Это исправляет важную границу предыдущего прохода.

Однако relation:

```text
Taxi Frontend
    → AUTHENTICATES_WITH
    → Core Backend
```

не должна быть повышена до `CONFIRMED`, пока exact frontend endpoint и exact backend handler не сопоставлены.

## 6. Next step

Выбрать один конкретный frontend auth function из раздела 2, установить его exact URL/request contract и найти ровно этот route в Core Backend. Затем проследить response → frontend state → следующий protected request.

