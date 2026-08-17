# Backend Semantic Graph — Research Pass 31
# Taxi Frontend Authentication Trace v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED
**Методология:** Semantic Graph Research Methodology v2.3
**Предшествующий проход:** RP-30 Driver Position → Passenger Map Consumer v0.1
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

## 1. Research Question

> Как конкретная версия Taxi Frontend получает authenticated context и как этот context передаётся в Core Backend API?

Authentication выбран следующим вертикальным блоком, поскольку он позволяет проверить одновременно:

```text
Frontend credential input
    ↓
auth state
    ↓
token / user identity
    ↓
API adapter
    ↓
Core Backend authentication
    ↓
role-aware frontend behavior
```

## 2. Найденные auth-related contexts

Всего auth/user/token/role contexts: **180**.

### `src/API/order.ts:22`
```text
17:   addToFormData, apiMethod, IApiMethodArguments, IResponseFields,
18: } from '../tools/api'
19: import Config from '../config'
20: import { t, TRANSLATION } from '../localization'
21: import store from '../state'
22: import { userSelectors } from '../state/user'
23: import { EBookingActions } from './constants'
24: import { getUserCar } from './car'
25: 
26: async function _postDrive(
27:   { formData }: IApiMethodArguments,
28:   data: IOrder,
29: ): Promise<IResponseFields & {
30:   b_id: IOrder['b_id'],
```

### `src/API/order.ts:128`
```text
123:   filter: {
124:     carClasses?: boolean
125:     locationClasses?: boolean
126:   } = {},
127: ) => {
128:   const userID = userSelectors.user(store.getState())?.u_id
129: 
130:   addToFormData(formData, {
131:     array_type: 'list',
132:   })
133: 
134:   const hiddenOrders = JSON.parse(localStorage.getItem('hiddenOrders') || '{}')
135:   const userHiddenOrders = hiddenOrders && userID && hiddenOrders[userID]
136: 
```

### `src/API/order.ts:172`
```text
167:   if (response.code !== '200')
168:     return response
169: 
170:   if (type === EOrderTypes.Active)
171:     for (const item of response.data.booking)
172:       item.user = response.data.user[item.u_id]
173: 
174:   return {
175:     ...response,
176:     data: {
177:       ...response.data,
178:       booking: response.data.booking
179:         ?.filter((item: IOrder) =>
180:           !(userHiddenOrders && userHiddenOrders.includes(item.b_id)),
```

### `src/API/order.ts:234`
```text
229:   /** Необходимое число машин поездки */
230:   b_cars_count: string,
231:   /** Если изменился статус поезки */
232:   b_state?: '1->2' | null
233: }> => {
234:   const userID = userSelectors.user(store.getState())?.u_id
235:   if (!userID) {
236:     Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
237:   }
238: 
239:   return getUserCar(userID as string)
240:     .then(car => {
241:       if (!car) return Promise.reject(t(TRANSLATION.NOT_LINKED_CAR))
242: 
```

### `src/API/order.ts:264`
```text
259: export const takeOrder = apiMethod<typeof _takeOrder>(_takeOrder)
260: 
261: const _chooseCandidate = (
262:   { formData }: IApiMethodArguments,
263:   id: IOrder['b_id'],
264:   user?: IUser['u_id'],
265: ): Promise<any> => {
266:   const userID = userSelectors.user(store.getState())?.u_id
267:   if (!userID) Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
268: 
269:   addToFormData(formData, {
270:     action: EBookingActions.SetPerformer,
271:     performer: '1',
272:     u_id: user,
```

### `src/API/order.ts:266`
```text
261: const _chooseCandidate = (
262:   { formData }: IApiMethodArguments,
263:   id: IOrder['b_id'],
264:   user?: IUser['u_id'],
265: ): Promise<any> => {
266:   const userID = userSelectors.user(store.getState())?.u_id
267:   if (!userID) Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
268: 
269:   addToFormData(formData, {
270:     action: EBookingActions.SetPerformer,
271:     performer: '1',
272:     u_id: user,
273:   })
274: 
```

### `src/API/order.ts:272`
```text
267:   if (!userID) Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
268: 
269:   addToFormData(formData, {
270:     action: EBookingActions.SetPerformer,
271:     performer: '1',
272:     u_id: user,
273:   })
274: 
275:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
276:     .then(res => res.data)
277:     .then(res => res.status === 'error' ? Promise.reject() : res)
278: }
279: export const chooseCandidate = apiMethod<typeof _chooseCandidate>(_chooseCandidate)
280: 
```

### `src/API/car.ts:35`
```text
30:   } & TCreateCar,
31: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
32:   const { u_id, ...formFields } = fields
33:   addToFormData(formData, { data: JSON.stringify(formFields) })
34:   const { data } =
35:     await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)
36:   return data
37: })
38: 
39: const _getCar = (
40:   { formData }: IApiMethodArguments,
41:   id: ICar['c_id'],
42: ): Promise<ICar | null> => {
43:   return axios.post(`${Config.API_URL}/car/${id}`, formData)
```

### `src/API/car.ts:95`
```text
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
```

### `src/API/car.ts:108`
```text
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
```

### `src/API/car.ts:144`
```text
139:     `${Config.API_URL}/car/${car.c_id}/drive`,
140:     formData,
141:   )
142:   if (
143:     response.code === '404' &&
144:     response.message === 'car is already driven by this user'
145:   ) response.data = { detail: 'not_modified' }
146: 
147:   if (
148:     response.code === '200' ||
149:     (response.code === '404' && response.data.detail === 'not_modified')
150:   ) {
151:     const syncUserResponse = await syncUserWithCar(car)
152:     if (!(syncUserResponse.code === '200' || (
```

### `src/API/car.ts:172`
```text
167:   addToFormData(formData, { data: JSON.stringify({
168:     'b_location_classes=': carClass?.booking_location_classes ??
169:       SITE_CONSTANTS.BOOKING_LOCATION_CLASSES.map(({ id }) => id),
170:   }) })
171:   const { data: response } =
172:     await axios.post(`${Config.API_URL}/user`, formData)
173:   if (
174:     response.code === '404' &&
175:     response.message === 'user or modified data not found'
176:   ) response.data = { detail: 'not_modified' }
177:   return response
178: })
179: 
180: async function _getUserDrivenCar(
```

### `src/API/car.ts:175`
```text
170:   }) })
171:   const { data: response } =
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
```

### `src/API/car.ts:184`
```text
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

### `src/API/user.ts:11`
```text
6: 
7: const _getUser = (
8:   { formData }: IApiMethodArguments,
9:   id: IUser['u_id'],
10: ): Promise<IUser | null> => {
11:   return axios.post(`${Config.API_URL}/user/${id}`, formData)
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
```

### `src/API/user.ts:13`
```text
8:   { formData }: IApiMethodArguments,
9:   id: IUser['u_id'],
10: ): Promise<IUser | null> => {
11:   return axios.post(`${Config.API_URL}/user/${id}`, formData)
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
```

### `src/API/user.ts:21`
```text
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
```

### `src/API/user.ts:23`
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
```

### `src/API/user.ts:30`
```text
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
```

### `src/API/user.ts:32`
```text
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
```

### `src/API/user.ts:36`
```text
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
```

### `src/API/user.ts:93`
```text
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
```

### `src/API/user.ts:98`
```text
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
```

### `src/API/user.ts:106`
```text
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

### `src/API/auth.ts:6`
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
```

### `src/API/auth.ts:56`
```text
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
```

### `src/API/auth.ts:60`
```text
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
```

### `src/API/auth.ts:66`
```text
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
```

### `src/API/auth.ts:69`
```text
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
```

### `src/API/auth.ts:71`
```text
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
```

### `src/API/auth.ts:78`
```text
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
```

### `src/API/auth.ts:85`
```text
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
```

### `src/API/auth.ts:92`
```text
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
```

### `src/API/auth.ts:105`
```text
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
```

### `src/API/auth.ts:108`
```text
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
```

### `src/API/auth.ts:110`
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
```

### `src/API/auth.ts:111`
```text
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
```

### `src/API/auth.ts:117`
```text
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
```

### `src/API/auth.ts:122`
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
```

### `src/API/auth.ts:129`
```text
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
```

### `src/API/auth.ts:144`
```text
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
```

### `src/API/auth.ts:157`
```text
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

### `src/API/auth.ts:158`
```text
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
```

### `src/API/auth.ts:159`
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
```

### `src/API/auth.ts:161`
```text
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

### `src/API/auth.ts:166`
```text
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
```

### `src/API/auth.ts:171`
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
```

### `src/API/auth.ts:172`
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
```

### `src/API/auth.ts:174`
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
```

### `src/API/auth.ts:178`
```text
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

### `src/API/auth.ts:180`
```text
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
```

### `src/API/auth.ts:181`
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
```

### `src/API/auth.ts:188`
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
```

### `src/API/auth.ts:189`
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
```

### `src/API/auth.ts:193`
```text
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
```

### `src/API/auth.ts:195`
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
```

### `src/API/auth.ts:196`
```text
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

### `src/API/auth.ts:207`
```text
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
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/way.ts:130`
```text
125:       to?: number,
126:     }
127:     const members: Members = {}
128:     const memberElements = relationElement.getElementsByTagName('member')
129:     for (const memberElement of memberElements) {
130:       const role = memberElement.getAttribute('role')!
131:       if (['from', 'via', 'to'].includes(role))
132:         members[role as keyof Members] =
133:           parseInt(memberElement.getAttribute('ref')!)
134:     }
135: 
136:     if (members.from && members.via && members.to && members.via in nodes) {
137:       const node = nodes[members.via]
138:       if (!node.turnRestrictions)
```

### `src/API/way.ts:131`
```text
126:     }
127:     const members: Members = {}
128:     const memberElements = relationElement.getElementsByTagName('member')
129:     for (const memberElement of memberElements) {
130:       const role = memberElement.getAttribute('role')!
131:       if (['from', 'via', 'to'].includes(role))
132:         members[role as keyof Members] =
133:           parseInt(memberElement.getAttribute('ref')!)
134:     }
135: 
136:     if (members.from && members.via && members.to && members.via in nodes) {
137:       const node = nodes[members.via]
138:       if (!node.turnRestrictions)
139:         node.turnRestrictions = []
```

### `src/API/way.ts:132`
```text
127:     const members: Members = {}
128:     const memberElements = relationElement.getElementsByTagName('member')
129:     for (const memberElement of memberElements) {
130:       const role = memberElement.getAttribute('role')!
131:       if (['from', 'via', 'to'].includes(role))
132:         members[role as keyof Members] =
133:           parseInt(memberElement.getAttribute('ref')!)
134:     }
135: 
136:     if (members.from && members.via && members.to && members.via in nodes) {
137:       const node = nodes[members.via]
138:       if (!node.turnRestrictions)
139:         node.turnRestrictions = []
140:       node.turnRestrictions!
```

### `src/API/index.ts:25`
```text
20: import getCountryISO3 from '../tools/countryISO2To3'
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
```

### `src/API/index.ts:33`
```text
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
```

### `src/API/index.ts:36`
```text
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
```

### `src/API/index.ts:37`
```text
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
```

### `src/API/index.ts:44`
```text
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
```

### `src/API/index.ts:85`
```text
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
```

### `src/API/index.ts:86`
```text
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
```

### `src/API/index.ts:233`
```text
228:           out_passengers: passengers,
229:         },
230:     ),
231:   })
232: 
233:   return axios.post(`${Config.API_URL}/user`, formData)
234:     .then(res => res.data)
235: }
236: export const setOutDrive = apiMethod<typeof _setOutDrive>(_setOutDrive)
237: 
238: export const reverseGeocode = (
239:   lat: ValueOf<Stringify<IBookingCoordinatesLatitude>>,
240:   lng: ValueOf<Stringify<IBookingCoordinatesLongitude>>,
241:   { details = true }: {
```

### `src/API/index.ts:320`
```text
315:       }
316:     })
317: }
318: 
319: export const notifyPosition = (point: IAddressPoint) => {
320:   const userID = userSelectors.user(store.getState())?.u_id
321: 
322:   axios.post('http://jecat.ru/car_api/api/notifypos.php', {
323:     driver: userID,
324:     lat: point.latitude,
325:     lon: point.longitude,
326:     time: new Date().getTime() / 1000,
327:   })
328: }
```

### `src/components/modals/LoginModal/Register.tsx:9`
```text
4: import Checkbox from '../../Checkbox'
5: import { getPhoneError } from '../../../tools/utils'
6: import { useForm, useWatch } from 'react-hook-form'
7: import Button from '../../Button'
8: import { IRootState } from '../../../state'
9: import { userSelectors, userActionCreators } from '../../../state/user'
10: import {
11:   ERegistrationType,
12:   LOGIN_TABS_IDS,
13: } from '../../../state/user/constants'
14: import { connect, ConnectedProps } from 'react-redux'
15: import cn from 'classnames'
16: import { EStatuses, EUserRoles, EWorkTypes, TFilesMap, IRequiredFields } from '../../../types/types'
17: import { useLocation } from 'react-router-dom'
```

### `src/components/modals/LoginModal/Register.tsx:13`
```text
8: import { IRootState } from '../../../state'
9: import { userSelectors, userActionCreators } from '../../../state/user'
10: import {
11:   ERegistrationType,
12:   LOGIN_TABS_IDS,
13: } from '../../../state/user/constants'
14: import { connect, ConnectedProps } from 'react-redux'
15: import cn from 'classnames'
16: import { EStatuses, EUserRoles, EWorkTypes, TFilesMap, IRequiredFields } from '../../../types/types'
17: import { useLocation } from 'react-router-dom'
18: import { yupResolver } from '@hookform/resolvers/yup'
19: import * as yup from 'yup'
20: import Alert from '../../Alert/Alert'
21: import { Intent } from '../../Alert'
```

### `src/components/modals/LoginModal/Register.tsx:29`
```text
24: import { normalizePhoneNumber } from '../../../tools/phoneUtils'
25: import { Input } from './elements'
26: 
27: const mapStateToProps = (state: IRootState) => {
28:   return {
29:     user: userSelectors.user(state),
30:     status: userSelectors.status(state),
31:     tab: userSelectors.tab(state),
32:     message: userSelectors.message(state),
33:     response: userSelectors.registerResponse(state),
34:   }
35: }
36: 
37: const mapDispatchToProps = {
```

### `src/components/modals/LoginModal/Register.tsx:328`
```text
323:   let isValidFrom = isValid
324:   if (isDriver && requireFeildsMap.passport_photo && !filesMap.passport_photo.length) isValidFrom = false
325:   if (isDriver && requireFeildsMap.driver_license_photo && !filesMap.driver_license_photo.length) isValidFrom = false
326:   if (isDriver && requireFeildsMap.license_photo && !filesMap.license_photo) isValidFrom = false
327:   return (
328:     <form className="login-form sign-up-subform" onSubmit={handleSubmit(onSubmit)}>
329:       {isDriver &&
330:         <Input
331:           inputProps={{
332:             onChange: (e: any) => setWorkType(Number(e.target.value)),
333:             value: String(workType),
334:           }}
335:           inputType={EInputTypes.Select}
336:           options={[
```

### `src/components/modals/LoginModal/Register.tsx:636`
```text
631: 
632:       <Button
633:         type="submit"
634:         text={t(TRANSLATION.SIGNUP)}
635:         fixedSize={false}
636:         className="login-modal_login-btn"
637:         skipHandler={true}
638:         disabled={!isValidFrom}
639:         status={status}
640:       />
641:     </form>
642:   )
643: }
644: 
```

### `src/components/modals/LoginModal/Login.tsx:11`
```text
6: import Button from '../../Button'
7: import { connect, ConnectedProps } from 'react-redux'
8: import images from '../../../constants/images'
9: import { IRootState } from '../../../state'
10: import { EStatuses, EUserRoles } from '../../../types/types'
11: import { userActionCreators, userSelectors } from '../../../state/user'
12: import { ERegistrationType, LOGIN_TABS_IDS } from '../../../state/user/constants'
13: import { emailRegex, phoneRegex } from '../../../tools/utils'
14: import { yupResolver } from '@hookform/resolvers/yup'
15: import * as yup from 'yup'
16: import Alert from '../../Alert/Alert'
17: import { Intent } from '../../Alert'
18: import { useVisibility } from '../../../tools/hooks'
19: import { GoogleLoginButton } from 'react-social-login-buttons'
```

### `src/components/modals/LoginModal/Login.tsx:12`
```text
7: import { connect, ConnectedProps } from 'react-redux'
8: import images from '../../../constants/images'
9: import { IRootState } from '../../../state'
10: import { EStatuses, EUserRoles } from '../../../types/types'
11: import { userActionCreators, userSelectors } from '../../../state/user'
12: import { ERegistrationType, LOGIN_TABS_IDS } from '../../../state/user/constants'
13: import { emailRegex, phoneRegex } from '../../../tools/utils'
14: import { yupResolver } from '@hookform/resolvers/yup'
15: import * as yup from 'yup'
16: import Alert from '../../Alert/Alert'
17: import { Intent } from '../../Alert'
18: import { useVisibility } from '../../../tools/hooks'
19: import { GoogleLoginButton } from 'react-social-login-buttons'
20: import { useLocation, useNavigate } from 'react-router-dom'
```

### `src/components/modals/LoginModal/Login.tsx:19`
```text
14: import { yupResolver } from '@hookform/resolvers/yup'
15: import * as yup from 'yup'
16: import Alert from '../../Alert/Alert'
17: import { Intent } from '../../Alert'
18: import { useVisibility } from '../../../tools/hooks'
19: import { GoogleLoginButton } from 'react-social-login-buttons'
20: import { useLocation, useNavigate } from 'react-router-dom'
21: import { modalsActionCreators,  modalsSelectors } from '../../../state/modals'
22: import { Input } from './elements'
23: 
24: 
25: const mapStateToProps = (state: IRootState) => ({
26:   user: userSelectors.user(state),
27:   status: userSelectors.status(state),
```

### `src/components/modals/LoginModal/Login.tsx:26`
```text
21: import { modalsActionCreators,  modalsSelectors } from '../../../state/modals'
22: import { Input } from './elements'
23: 
24: 
25: const mapStateToProps = (state: IRootState) => ({
26:   user: userSelectors.user(state),
27:   status: userSelectors.status(state),
28:   tab: userSelectors.tab(state),
29:   message: userSelectors.message(state),
30:   isWAOpen: modalsSelectors.isWACodeModalOpen,
31: })
32: 
33: const mapDispatchToProps = {
34:   login: userActionCreators.login,
```

### `src/components/modals/LoginModal/Login.tsx:34`
```text
29:   message: userSelectors.message(state),
30:   isWAOpen: modalsSelectors.isWACodeModalOpen,
31: })
32: 
33: const mapDispatchToProps = {
34:   login: userActionCreators.login,
35:   setLoginModal: modalsActionCreators.setLoginModal,
36:   googleLogin: userActionCreators.googleLogin,
37:   logout: userActionCreators.logout,
38:   remindPassword: userActionCreators.remindPassword,
39:   setStatus: userActionCreators.setStatus,
40:   setMessage: userActionCreators.setMessage,
41:   register: userActionCreators.register,
42:   setWAOpen: modalsActionCreators.setWACodeModal,
```

### `src/components/modals/LoginModal/Login.tsx:37`
```text
32: 
33: const mapDispatchToProps = {
34:   login: userActionCreators.login,
35:   setLoginModal: modalsActionCreators.setLoginModal,
36:   googleLogin: userActionCreators.googleLogin,
37:   logout: userActionCreators.logout,
38:   remindPassword: userActionCreators.remindPassword,
39:   setStatus: userActionCreators.setStatus,
40:   setMessage: userActionCreators.setMessage,
41:   register: userActionCreators.register,
42:   setWAOpen: modalsActionCreators.setWACodeModal,
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
44: }
45: 
```

### `src/components/modals/LoginModal/Login.tsx:49`
```text
44: }
45: 
46: const connector = connect(mapStateToProps, mapDispatchToProps)
47: 
48: interface IFormValues {
49:     login: string | undefined,
50:     password?: string | undefined,
51:     type: ERegistrationType
52: }
53: 
54: interface IProps extends ConnectedProps<typeof connector> {
55:     isOpen: boolean,
56: }
57: 
```

### `src/components/modals/LoginModal/Login.tsx:60`
```text
55:     isOpen: boolean,
56: }
57: 
58: 
59: const LoginForm: React.FC<IProps> = ({
60:   user,
61:   status,
62:   tab,
63:   googleLogin,
64:   isOpen,
65:   setWAOpen,
66:   setRefOpen,
67:   login,
68:   logout,
```

### `src/components/modals/LoginModal/Login.tsx:67`
```text
62:   tab,
63:   googleLogin,
64:   isOpen,
65:   setWAOpen,
66:   setRefOpen,
67:   login,
68:   logout,
69:   remindPassword,
70:   setMessage,
71:   setStatus,
72:   setLoginModal,
73:   message,
74: }) => {
75:   const [isPasswordShows, setIsPasswordShows] = useState(false)
```

### `src/components/modals/LoginModal/Login.tsx:68`
```text
63:   googleLogin,
64:   isOpen,
65:   setWAOpen,
66:   setRefOpen,
67:   login,
68:   logout,
69:   remindPassword,
70:   setMessage,
71:   setStatus,
72:   setLoginModal,
73:   message,
74: }) => {
75:   const [isPasswordShows, setIsPasswordShows] = useState(false)
76:   const [dataToLogin, setDataToLogin] = useState({})
```

### `src/components/modals/LoginModal/Login.tsx:83`
```text
78:   const [isPasswordVisible, togglePasswordVisibility] = useVisibility(true)
79:   const location = useLocation()
80:   const navigate = useNavigate()
81:   const googleClientId = '973943716904-b33r11ijgi08m5etsg5ndv409shh1tjl.apps.googleusercontent.com'
82: 
83:   const role = !location.pathname.includes('/driver-order') ?
84:     EUserRoles.Client :
85:     EUserRoles.Driver
86: 
87:   const schema = yup.object({
88:     type: yup.string().required(),
89:     login: yup.string().required().when('type', {
90:       is: (type: ERegistrationType) => (type === ERegistrationType.Email),
91:       then: yup.string().required().matches(emailRegex, t(TRANSLATION.EMAIL_ERROR)),
```

### `src/components/modals/LoginModal/Login.tsx:89`
```text
84:     EUserRoles.Client :
85:     EUserRoles.Driver
86: 
87:   const schema = yup.object({
88:     type: yup.string().required(),
89:     login: yup.string().required().when('type', {
90:       is: (type: ERegistrationType) => (type === ERegistrationType.Email),
91:       then: yup.string().required().matches(emailRegex, t(TRANSLATION.EMAIL_ERROR)),
92:       otherwise: yup.string().required().matches(phoneRegex, t(TRANSLATION.PHONE_PATTERN_ERROR)),
93:     }),
94:   })
95: 
96: 
97:   const {
```

### `src/components/modals/LoginModal/Login.tsx:107`
```text
102:     trigger,
103:   } = useForm<IFormValues>({
104:     criteriaMode: 'all',
105:     mode: 'all',
106:     defaultValues: {
107:       login: user?.u_email || '',
108:       type: ERegistrationType.Email,
109:     },
110:     resolver: yupResolver(user ? yup.object() : schema),
111:   })
112:   const { login: formLogin, type } = useWatch<IFormValues>({ control })
113: 
114:   useEffect(() => {
115:     if (!isOpen) {
```

### `src/components/modals/LoginModal/Login.tsx:110`
```text
105:     mode: 'all',
106:     defaultValues: {
107:       login: user?.u_email || '',
108:       type: ERegistrationType.Email,
109:     },
110:     resolver: yupResolver(user ? yup.object() : schema),
111:   })
112:   const { login: formLogin, type } = useWatch<IFormValues>({ control })
113: 
114:   useEffect(() => {
115:     if (!isOpen) {
116:       setStatus(EStatuses.Default)
117:       setMessage('')
118:     }
```

### `src/components/modals/LoginModal/Login.tsx:112`
```text
107:       login: user?.u_email || '',
108:       type: ERegistrationType.Email,
109:     },
110:     resolver: yupResolver(user ? yup.object() : schema),
111:   })
112:   const { login: formLogin, type } = useWatch<IFormValues>({ control })
113: 
114:   useEffect(() => {
115:     if (!isOpen) {
116:       setStatus(EStatuses.Default)
117:       setMessage('')
118:     }
119:     if (isOpen && isVisible) {
120:       toggleVisibility()
```

### `src/components/modals/LoginModal/Login.tsx:160`
```text
155:   }, [type])
156: 
157: 
158:   useEffect(() => {
159:     if (
160:       (status === EStatuses.Fail || status === EStatuses.Success && user) &&
161:       type !== ERegistrationType.Whatsapp && !isVisible
162:     ) {
163:       console.log('togglin, prev: ', isVisible)
164:       toggleVisibility()
165:     } else if (status === EStatuses.Whatsapp) {
166:       setLoginModal(false)
167:       setWAOpen({
168:         isOpen: true,
```

### `src/components/modals/LoginModal/Login.tsx:169`
```text
164:       toggleVisibility()
165:     } else if (status === EStatuses.Whatsapp) {
166:       setLoginModal(false)
167:       setWAOpen({
168:         isOpen: true,
169:         login: login,
170:         data: { ...dataToLogin, navigate },
171:       })
172:     }
173:   }, [status])
174:   useEffect(() => {
175:     if(status === EStatuses.Success && message==='remind_password_success') {
176:       toggleVisibility()
177:       if (!isVisible) {
```

### `src/components/modals/LoginModal/Login.tsx:186`
```text
181:   }, [status])
182: 
183:   if (tab !== LOGIN_TABS_IDS[0]) return null
184: 
185:   const onSubmit = (data: IFormValues) => {
186:     console.log(status, user)
187:     if (isVisible) toggleVisibility()
188:     setDataToLogin(data)
189:     if (user) {
190:       logout()
191:     } else if (data) {
192:       const loginData: IFormValues = data.type === 'whatsapp' ?
193:         {
194:           type: data.type,
```

### `src/components/modals/LoginModal/Login.tsx:189`
```text
184: 
185:   const onSubmit = (data: IFormValues) => {
186:     console.log(status, user)
187:     if (isVisible) toggleVisibility()
188:     setDataToLogin(data)
189:     if (user) {
190:       logout()
191:     } else if (data) {
192:       const loginData: IFormValues = data.type === 'whatsapp' ?
193:         {
194:           type: data.type,
195:           login: data.login || '',
196:         } :
197:         {
```

### `src/components/modals/LoginModal/Login.tsx:190`
```text
185:   const onSubmit = (data: IFormValues) => {
186:     console.log(status, user)
187:     if (isVisible) toggleVisibility()
188:     setDataToLogin(data)
189:     if (user) {
190:       logout()
191:     } else if (data) {
192:       const loginData: IFormValues = data.type === 'whatsapp' ?
193:         {
194:           type: data.type,
195:           login: data.login || '',
196:         } :
197:         {
198:           ...data,
```

### `src/components/modals/LoginModal/Login.tsx:195`
```text
190:       logout()
191:     } else if (data) {
192:       const loginData: IFormValues = data.type === 'whatsapp' ?
193:         {
194:           type: data.type,
195:           login: data.login || '',
196:         } :
197:         {
198:           ...data,
199:           login: data.login || '',
200:         }
201:       login({ ...loginData, navigate: navigate })
202:     }
203:   }
```

### `src/components/modals/LoginModal/Login.tsx:199`
```text
194:           type: data.type,
195:           login: data.login || '',
196:         } :
197:         {
198:           ...data,
199:           login: data.login || '',
200:         }
201:       login({ ...loginData, navigate: navigate })
202:     }
203:   }
204: 
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
207:     return value && decodeURIComponent(value)
```

### `src/components/modals/LoginModal/Login.tsx:201`
```text
196:         } :
197:         {
198:           ...data,
199:           login: data.login || '',
200:         }
201:       login({ ...loginData, navigate: navigate })
202:     }
203:   }
204: 
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
207:     return value && decodeURIComponent(value)
208:   }
209: 
```

### `src/components/modals/LoginModal/Login.tsx:212`
```text
207:     return value && decodeURIComponent(value)
208:   }
209: 
210:   return (
211:     <form
212:       className="login-form sign-in-subform"
213:       onSubmit={handleSubmit(onSubmit)}
214:     >
215:       <Input
216:         inputProps={{
217:           ...formRegister('login'),
218:           placeholder: type === ERegistrationType.Phone || type === ERegistrationType.Whatsapp ?
219:             t(TRANSLATION.PHONE) :
220:             t(TRANSLATION.EMAIL),
```

### `src/components/modals/LoginModal/Login.tsx:217`
```text
212:       className="login-form sign-in-subform"
213:       onSubmit={handleSubmit(onSubmit)}
214:     >
215:       <Input
216:         inputProps={{
217:           ...formRegister('login'),
218:           placeholder: type === ERegistrationType.Phone || type === ERegistrationType.Whatsapp ?
219:             t(TRANSLATION.PHONE) :
220:             t(TRANSLATION.EMAIL),
221:         }}
222:         label={t(TRANSLATION.LOGIN)}
223:         error={errors.login?.message}
224:         key={type}
225:       />
```

### `src/components/modals/LoginModal/Login.tsx:222`
```text
217:           ...formRegister('login'),
218:           placeholder: type === ERegistrationType.Phone || type === ERegistrationType.Whatsapp ?
219:             t(TRANSLATION.PHONE) :
220:             t(TRANSLATION.EMAIL),
221:         }}
222:         label={t(TRANSLATION.LOGIN)}
223:         error={errors.login?.message}
224:         key={type}
225:       />
226:       {isPasswordVisible &&
227:             <Input
228:               inputProps={{
229:                 ...formRegister('password'),
230:                 type: isPasswordShows ? 'text' : 'password',
```

### `src/components/modals/LoginModal/Login.tsx:223`
```text
218:           placeholder: type === ERegistrationType.Phone || type === ERegistrationType.Whatsapp ?
219:             t(TRANSLATION.PHONE) :
220:             t(TRANSLATION.EMAIL),
221:         }}
222:         label={t(TRANSLATION.LOGIN)}
223:         error={errors.login?.message}
224:         key={type}
225:       />
226:       {isPasswordVisible &&
227:             <Input
228:               inputProps={{
229:                 ...formRegister('password'),
230:                 type: isPasswordShows ? 'text' : 'password',
231:                 placeholder: t(TRANSLATION.PASSWORD),
```

### `src/components/modals/LoginModal/Login.tsx:241`
```text
236:                 {
237:                   src: isPasswordShows ? images.closedEye : images.openedEye,
238:                   onClick: () => setIsPasswordShows(prev => !prev),
239:                 },
240:                 {
241:                   ...(!user ?
242:                     {
243:                       className: 'restore-password-block__button',
244:                       type: 'button',
245:                       onClick: () => {
246:                         formLogin && window.confirm(t(TRANSLATION.PASSWORD_RESET_MESSAGE)) && remindPassword(formLogin)
247:                       },
248:                       disabled: !formLogin || !!errors?.login,
249:                       text: t(TRANSLATION.RESTORE_PASSWORD),
```

### `src/components/modals/LoginModal/Login.tsx:248`
```text
243:                       className: 'restore-password-block__button',
244:                       type: 'button',
245:                       onClick: () => {
246:                         formLogin && window.confirm(t(TRANSLATION.PASSWORD_RESET_MESSAGE)) && remindPassword(formLogin)
247:                       },
248:                       disabled: !formLogin || !!errors?.login,
249:                       text: t(TRANSLATION.RESTORE_PASSWORD),
250:                       skipHandler: true,
251:                     } :
252:                     {}
253:                   ),
254:                 },
255:               ].filter(item => Object.values(item).length)}
256:             />
```

### `src/components/modals/LoginModal/Login.tsx:285`
```text
280:                 onClose={toggleVisibility}
281:               />
282:             </div>
283:       }
284: 
285:       {Number(role) !== EUserRoles.Driver && (
286:         // <LoginSocialGoogle
287:         //   client_id={googleClientId}
288:         //   onLoginStart={() => {}}
289:         //   redirect_uri={''}
290:         //   scope="profile email"
291:         //   access_type="online"
292:         //   onResolve={(data) => {
293:         //     console.log(data)
```

### `src/components/modals/LoginModal/Login.tsx:310`
```text
305:         //   }}
306:         //   onReject={err => {
307:         //     console.log(err)
308:         //   }}
309:         // >
310:         <a href={`https://accounts.google.com/o/oauth2/v2/auth?response_type=code&access_type=offline&client_id=${googleClientId}&redirect_uri=${Config.SERVER_URL}/google/&state&scope=email%20profile&prompt=select_account`}>
311:           <GoogleLoginButton />
312:         </a>
313: 
314:       )}
315: 
316:       <Button
317:         type="submit"
318:         text={!!user ? t(TRANSLATION.LOGOUT) : t(TRANSLATION.SIGN_IN)}
```

### `src/components/modals/LoginModal/Login.tsx:318`
```text
313: 
314:       )}
315: 
316:       <Button
317:         type="submit"
318:         text={!!user ? t(TRANSLATION.LOGOUT) : t(TRANSLATION.SIGN_IN)}
319:         fixedSize={false}
320:         className="login-modal_login-btn"
321:         skipHandler={true}
322:         disabled={!!Object.values(errors).length}
323:         status={status}
324:       />
325:     </form>
326:   )
```

### `src/components/modals/LoginModal/Login.tsx:320`
```text
315: 
316:       <Button
317:         type="submit"
318:         text={!!user ? t(TRANSLATION.LOGOUT) : t(TRANSLATION.SIGN_IN)}
319:         fixedSize={false}
320:         className="login-modal_login-btn"
321:         skipHandler={true}
322:         disabled={!!Object.values(errors).length}
323:         status={status}
324:       />
325:     </form>
326:   )
327: }
328: 
```

### `src/components/modals/LoginModal/RegisterJSON.tsx:6`
```text
1: import React, { useMemo } from 'react'
2: import { connect, ConnectedProps } from 'react-redux'
3: import { EStatuses, EUserRoles } from '../../../types/types'
4: import { normalizePhoneNumber } from '../../../tools/phoneUtils'
5: import { IRootState } from '../../../state'
6: import { userSelectors, userActionCreators } from '../../../state/user'
7: import { t, TRANSLATION } from '../../../localization'
8: import JSONForm from '../../JSONForm'
9: import { TForm } from '../../JSONForm/types'
10: import ErrorFrame from '../../ErrorFrame'
11: 
12: const mapStateToProps = (state: IRootState) => {
13:   return {
14:     user: userSelectors.user(state),
```

### `src/components/modals/LoginModal/RegisterJSON.tsx:14`
```text
9: import { TForm } from '../../JSONForm/types'
10: import ErrorFrame from '../../ErrorFrame'
11: 
12: const mapStateToProps = (state: IRootState) => {
13:   return {
14:     user: userSelectors.user(state),
15:     status: userSelectors.status(state),
16:     tab: userSelectors.tab(state),
17:     message: userSelectors.message(state),
18:     response: userSelectors.registerResponse(state),
19:   }
20: }
21: 
22: const mapDispatchToProps = {
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:9`
```text
4: import Button from '../../Button'
5: import * as API from '../../../API'
6: import '../styles.scss'
7: import { IRootState } from '../../../state'
8: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
9: import { userActionCreators } from '../../../state/user'
10: import Overlay from '../Overlay'
11: import { userSelectors } from '../../../state/user'
12: import Input from '../../Input'
13: import { defaultRefCodeModal } from '../../../state/modals/reducer'
14: import * as yup from 'yup'
15: import { useForm } from 'react-hook-form'
16: import { yupResolver } from '@hookform/resolvers/yup'
17: import { t, TRANSLATION } from '../../../localization'
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:11`
```text
6: import '../styles.scss'
7: import { IRootState } from '../../../state'
8: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
9: import { userActionCreators } from '../../../state/user'
10: import Overlay from '../Overlay'
11: import { userSelectors } from '../../../state/user'
12: import Input from '../../Input'
13: import { defaultRefCodeModal } from '../../../state/modals/reducer'
14: import * as yup from 'yup'
15: import { useForm } from 'react-hook-form'
16: import { yupResolver } from '@hookform/resolvers/yup'
17: import { t, TRANSLATION } from '../../../localization'
18: import { EStatuses } from '../../../types/types'
19: import { useVisibility } from '../../../tools/hooks'
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:22`
```text
17: import { t, TRANSLATION } from '../../../localization'
18: import { EStatuses } from '../../../types/types'
19: import { useVisibility } from '../../../tools/hooks'
20: import Alert from '../../Alert/Alert'
21: import { Intent } from '../../Alert'
22: import { ERegistrationType } from '../../../state/user/constants'
23: interface IFormValues {
24:   code: string,
25: }
26: 
27: const mapStateToProps = (state: IRootState) => ({
28:   payload: modalsSelectors.isRefCodeModalOpen(state),
29:   user: userSelectors.user(state),
30:   whatsappSignUpData: userSelectors.whatsappSignUpData(state),
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:29`
```text
24:   code: string,
25: }
26: 
27: const mapStateToProps = (state: IRootState) => ({
28:   payload: modalsSelectors.isRefCodeModalOpen(state),
29:   user: userSelectors.user(state),
30:   whatsappSignUpData: userSelectors.whatsappSignUpData(state),
31:   status: userSelectors.status(state),
32: })
33: 
34: const mapDispatchToProps = {
35:   googleLogin: userActionCreators.googleLogin,
36:   login: userActionCreators.login,
37:   whatsappSignUp: userActionCreators.whatsappSignUp,
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:36`
```text
31:   status: userSelectors.status(state),
32: })
33: 
34: const mapDispatchToProps = {
35:   googleLogin: userActionCreators.googleLogin,
36:   login: userActionCreators.login,
37:   whatsappSignUp: userActionCreators.whatsappSignUp,
38:   setRefCodeModal: modalsActionCreators.setRefCodeModal,
39:   setCancelModal: modalsActionCreators.setCancelModal,
40: }
41: 
42: const connector = connect(mapStateToProps, mapDispatchToProps)
43: 
44: interface IProps extends ConnectedProps<typeof connector> {
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:91`
```text
86:     let data = payload.data
87:     if (!formData.code) {
88:       if(whatsappSignUpData?.u_phone) {
89:         whatsappSignUp({
90:           type: ERegistrationType.Whatsapp,
91:           login: whatsappSignUpData.u_phone,
92:           navigate,
93:         })
94:         return
95:       } else {
96:         googleLogin({ data, auth_hash: null, navigate })
97:         return
98:       }
99:     }
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:111`
```text
106:       data.ref_code = formData.code
107: 
108:       if(whatsappSignUpData?.u_phone) {
109:         whatsappSignUp({
110:           type: ERegistrationType.Whatsapp,
111:           login: whatsappSignUpData.u_phone,
112:           ref_code: formData.code,
113:           navigate,
114:         })
115:         return
116:       } else {
117:         googleLogin({ data, auth_hash: null, navigate })
118:         return
119:       }
```

### `src/components/modals/LoginModal/LogoutForm.tsx:7`
```text
2: import { connect, ConnectedProps } from 'react-redux'
3: import { IRootState } from '../../../state'
4: import {
5:   userSelectors,
6:   userActionCreators,
7: } from '../../../state/user'
8: import { t, TRANSLATION } from '../../../localization'
9: import { EInputTypes } from '../../Input'
10: import Button from '../../Button'
11: import { Input } from './elements'
12: import './styles.scss'
13: 
14: const mapStateToProps = (state: IRootState) => ({
15:   user: userSelectors.user(state),
```

### `src/components/modals/LoginModal/LogoutForm.tsx:15`
```text
10: import Button from '../../Button'
11: import { Input } from './elements'
12: import './styles.scss'
13: 
14: const mapStateToProps = (state: IRootState) => ({
15:   user: userSelectors.user(state),
16: })
17: 
18: const mapDispatchToProps = {
19:   logout: userActionCreators.logout,
20: }
21: 
22: const connector = connect(mapStateToProps, mapDispatchToProps)
23: 
```

### `src/components/modals/LoginModal/LogoutForm.tsx:19`
```text
14: const mapStateToProps = (state: IRootState) => ({
15:   user: userSelectors.user(state),
16: })
17: 
18: const mapDispatchToProps = {
19:   logout: userActionCreators.logout,
20: }
21: 
22: const connector = connect(mapStateToProps, mapDispatchToProps)
23: 
24: interface IProps extends ConnectedProps<typeof connector> {}
25: 
26: function LogoutForm({
27:   user,
```

### `src/components/modals/LoginModal/LogoutForm.tsx:27`
```text
22: const connector = connect(mapStateToProps, mapDispatchToProps)
23: 
24: interface IProps extends ConnectedProps<typeof connector> {}
25: 
26: function LogoutForm({
27:   user,
28:   logout,
29: }: IProps) {
30:   return user && (
31:     <form
32:       className="login-form sign-in-subform"
33:       onSubmit={event => {
34:         event.preventDefault()
35:       }}
```

### `src/components/modals/LoginModal/LogoutForm.tsx:28`
```text
23: 
24: interface IProps extends ConnectedProps<typeof connector> {}
25: 
26: function LogoutForm({
27:   user,
28:   logout,
29: }: IProps) {
30:   return user && (
31:     <form
32:       className="login-form sign-in-subform"
33:       onSubmit={event => {
34:         event.preventDefault()
35:       }}
36:     >
```

### `src/components/modals/LoginModal/LogoutForm.tsx:30`
```text
25: 
26: function LogoutForm({
27:   user,
28:   logout,
29: }: IProps) {
30:   return user && (
31:     <form
32:       className="login-form sign-in-subform"
33:       onSubmit={event => {
34:         event.preventDefault()
35:       }}
36:     >
37:       <Input
38:         inputProps={{
```

### `src/components/modals/LoginModal/LogoutForm.tsx:32`
```text
27:   user,
28:   logout,
29: }: IProps) {
30:   return user && (
31:     <form
32:       className="login-form sign-in-subform"
33:       onSubmit={event => {
34:         event.preventDefault()
35:       }}
36:     >
37:       <Input
38:         inputProps={{
39:           disabled: true,
40:           value: user.u_phone ?? user.u_email,
```

### `src/components/modals/LoginModal/LogoutForm.tsx:40`
```text
35:       }}
36:     >
37:       <Input
38:         inputProps={{
39:           disabled: true,
40:           value: user.u_phone ?? user.u_email,
41:         }}
42:         inputType={user.u_phone ? EInputTypes.MaskedPhone : EInputTypes.Default}
43:         label={t(TRANSLATION.LOGIN)}
44:       />
45: 
46:       <Button
47:         type="submit"
48:         text={t(TRANSLATION.LOGOUT)}
```

### `src/components/modals/LoginModal/LogoutForm.tsx:42`
```text
37:       <Input
38:         inputProps={{
39:           disabled: true,
40:           value: user.u_phone ?? user.u_email,
41:         }}
42:         inputType={user.u_phone ? EInputTypes.MaskedPhone : EInputTypes.Default}
43:         label={t(TRANSLATION.LOGIN)}
44:       />
45: 
46:       <Button
47:         type="submit"
48:         text={t(TRANSLATION.LOGOUT)}
49:         fixedSize={false}
50:         className="login-modal_login-btn"
```

### `src/components/modals/LoginModal/LogoutForm.tsx:43`
```text
38:         inputProps={{
39:           disabled: true,
40:           value: user.u_phone ?? user.u_email,
41:         }}
42:         inputType={user.u_phone ? EInputTypes.MaskedPhone : EInputTypes.Default}
43:         label={t(TRANSLATION.LOGIN)}
44:       />
45: 
46:       <Button
47:         type="submit"
48:         text={t(TRANSLATION.LOGOUT)}
49:         fixedSize={false}
50:         className="login-modal_login-btn"
51:         skipHandler={true}
```

### `src/components/modals/LoginModal/LogoutForm.tsx:48`
```text
43:         label={t(TRANSLATION.LOGIN)}
44:       />
45: 
46:       <Button
47:         type="submit"
48:         text={t(TRANSLATION.LOGOUT)}
49:         fixedSize={false}
50:         className="login-modal_login-btn"
51:         skipHandler={true}
52:         onClick={() => {
53:           logout()
54:         }}
55:       />
56:     </form>
```

### `src/components/modals/LoginModal/LogoutForm.tsx:50`
```text
45: 
46:       <Button
47:         type="submit"
48:         text={t(TRANSLATION.LOGOUT)}
49:         fixedSize={false}
50:         className="login-modal_login-btn"
51:         skipHandler={true}
52:         onClick={() => {
53:           logout()
54:         }}
55:       />
56:     </form>
57:   )
58: }
```

### `src/components/modals/LoginModal/LogoutForm.tsx:53`
```text
48:         text={t(TRANSLATION.LOGOUT)}
49:         fixedSize={false}
50:         className="login-modal_login-btn"
51:         skipHandler={true}
52:         onClick={() => {
53:           logout()
54:         }}
55:       />
56:     </form>
57:   )
58: }
59: 
60: export default connector(LogoutForm)
```

### `src/components/modals/LoginModal/elements.tsx:11`
```text
6:   fieldWrapperClassName,
7:   ...props
8: }: React.ComponentProps<typeof BaseInput>) {
9:   return (
10:     <BaseInput
11:       style={EInputStyles.Login}
12:       fieldWrapperClassName={cn('login-form__input', fieldWrapperClassName)}
13:       {...props}
14:     />
15:   )
16: }
```

### `src/components/modals/LoginModal/elements.tsx:12`
```text
7:   ...props
8: }: React.ComponentProps<typeof BaseInput>) {
9:   return (
10:     <BaseInput
11:       style={EInputStyles.Login}
12:       fieldWrapperClassName={cn('login-form__input', fieldWrapperClassName)}
13:       {...props}
14:     />
15:   )
16: }
```

### `src/components/modals/LoginModal/index.tsx:9`
```text
4: import { EStatuses } from '../../../types/types'
5: import { IRootState } from '../../../state'
6: import {
7:   userSelectors,
8:   userActionCreators,
9: } from '../../../state/user'
10: import { LOGIN_TABS } from '../../../state/user/constants'
11: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
12: import { t, TRANSLATION } from '../../../localization'
13: import Tabs from '../../tabs/Tabs'
14: import VersionInfo from '../../version-info'
15: import LoadFrame from '../../LoadFrame'
16: import Overlay from '../Overlay'
17: import LoginForm from './Login'
```

### `src/components/modals/LoginModal/index.tsx:10`
```text
5: import { IRootState } from '../../../state'
6: import {
7:   userSelectors,
8:   userActionCreators,
9: } from '../../../state/user'
10: import { LOGIN_TABS } from '../../../state/user/constants'
11: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
12: import { t, TRANSLATION } from '../../../localization'
13: import Tabs from '../../tabs/Tabs'
14: import VersionInfo from '../../version-info'
15: import LoadFrame from '../../LoadFrame'
16: import Overlay from '../Overlay'
17: import LoginForm from './Login'
18: import RegisterForm from './Register'
```

### `src/components/modals/LoginModal/index.tsx:17`
```text
12: import { t, TRANSLATION } from '../../../localization'
13: import Tabs from '../../tabs/Tabs'
14: import VersionInfo from '../../version-info'
15: import LoadFrame from '../../LoadFrame'
16: import Overlay from '../Overlay'
17: import LoginForm from './Login'
18: import RegisterForm from './Register'
19: import LogoutForm from './LogoutForm'
20: import RegisterJSON from './RegisterJSON'
21: import './styles.scss'
22: 
23: const mapStateToProps = (state: IRootState) => ({
24:   isOpen: modalsSelectors.isLoginModalOpen(state),
25:   user: userSelectors.user(state),
```

### `src/components/modals/LoginModal/index.tsx:25`
```text
20: import RegisterJSON from './RegisterJSON'
21: import './styles.scss'
22: 
23: const mapStateToProps = (state: IRootState) => ({
24:   isOpen: modalsSelectors.isLoginModalOpen(state),
25:   user: userSelectors.user(state),
26:   status: userSelectors.status(state),
27:   tab: userSelectors.tab(state),
28: })
29: 
30: const mapDispatchToProps = {
31:   setLoginModal: modalsActionCreators.setLoginModal,
32:   setTab: userActionCreators.setTab,
33: }
```

### `src/components/modals/LoginModal/index.tsx:41`
```text
36: 
37: interface IProps extends ConnectedProps<typeof connector> {}
38: 
39: function LoginModal({
40:   isOpen,
41:   user,
42:   status,
43:   tab,
44:   setTab,
45:   setLoginModal,
46: }: IProps) {
47:   const location = useLocation()
48:   const _TABS = user ?
49:     [] :
```

### `src/components/modals/LoginModal/index.tsx:48`
```text
43:   tab,
44:   setTab,
45:   setLoginModal,
46: }: IProps) {
47:   const location = useLocation()
48:   const _TABS = user ?
49:     [] :
50:     LOGIN_TABS.map((item, index) => ({
51:       ...item,
52:       label: t(item.label),
53:     }))
54: 
55:   const RegisterComponent = location.pathname.includes('/driver-order') ?
56:     RegisterJSON :
```

### `src/components/modals/LoginModal/index.tsx:63`
```text
58:   return (
59:     <Overlay
60:       isOpen={isOpen}
61:       onClick={() => setLoginModal(false)}
62:     >
63:       <div className="modal login-modal">
64:         {status === EStatuses.Loading && <LoadFrame />}
65: 
66:         <fieldset>
67:           <legend>
68:             {user ?
69:               t(TRANSLATION.PROFILE) :
70:               tab === 'sign-in' ?
71:                 t(TRANSLATION.SIGN_IN_HEADER) :
```

### `src/components/modals/LoginModal/index.tsx:68`
```text
63:       <div className="modal login-modal">
64:         {status === EStatuses.Loading && <LoadFrame />}
65: 
66:         <fieldset>
67:           <legend>
68:             {user ?
69:               t(TRANSLATION.PROFILE) :
70:               tab === 'sign-in' ?
71:                 t(TRANSLATION.SIGN_IN_HEADER) :
72:                 t(TRANSLATION.SIGNUP)
73:             }
74:           </legend>
75: 
76:           <div className="login">
```

### `src/components/modals/LoginModal/index.tsx:76`
```text
71:                 t(TRANSLATION.SIGN_IN_HEADER) :
72:                 t(TRANSLATION.SIGNUP)
73:             }
74:           </legend>
75: 
76:           <div className="login">
77:             {useMemo(() => _TABS.length > 0 &&
78:               <Tabs
79:                 tabs={_TABS}
80:                 activeTabID={tab}
81:                 onChange={id => setTab(id as typeof tab)}
82:               />
83:             , [_TABS, tab, setTab])}
84: 
```

### `src/components/modals/LoginModal/index.tsx:85`
```text
80:                 activeTabID={tab}
81:                 onChange={id => setTab(id as typeof tab)}
82:               />
83:             , [_TABS, tab, setTab])}
84: 
85:             {useMemo(() => user ?
86:               <LogoutForm /> :
87:               tab === 'sign-in' ?
88:                 <LoginForm
89:                   isOpen={isOpen}
90:                 /> :
91:                 <RegisterComponent
92:                   isOpen={isOpen}
93:                 />
```

### `src/components/modals/LoginModal/index.tsx:94`
```text
89:                   isOpen={isOpen}
90:                 /> :
91:                 <RegisterComponent
92:                   isOpen={isOpen}
93:                 />
94:             , [user, tab, isOpen])}
95:           </div>
96:         </fieldset>
97:         <VersionInfo/>
98:       </div>
99:     </Overlay>
100:   )
101: }
102: 
```

### `src/components/modals/LoginModal/WACodeModal.tsx:8`
```text
3: import Button from '../../Button'
4: import '../styles.scss'
5: import { IRootState } from '../../../state'
6: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
7: import Overlay from '../Overlay'
8: import { userSelectors } from '../../../state/user'
9: import Input from '../../Input'
10: import { defaultWACodeModal } from '../../../state/modals/reducer'
11: import * as yup from 'yup'
12: import { useForm, useWatch } from 'react-hook-form'
13: import { yupResolver } from '@hookform/resolvers/yup'
14: import { t, TRANSLATION } from '../../../localization'
15: import { EStatuses } from '../../../types/types'
16: import { useVisibility } from '../../../tools/hooks'
```

### `src/components/modals/LoginModal/WACodeModal.tsx:27`
```text
22:   code: number,
23: }
24: 
25: const mapStateToProps = (state: IRootState) => ({
26:   payload: modalsSelectors.isWACodeModalOpen(state),
27:   user: userSelectors.user(state),
28:   status: userSelectors.status(state),
29: })
30: 
31: const mapDispatchToProps = {
32:   setWACodeModal: modalsActionCreators.setWACodeModal,
33:   setCancelModal: modalsActionCreators.setCancelModal,
34: 
35: }
```

### `src/components/modals/LoginModal/WACodeModal.tsx:78`
```text
73:   })
74: 
75:   const onSubmit = (formData : IFormValues) => {
76:     let data = payload.data
77:     data.password = formData.code
78:     payload.login(data)
79: 
80:   }
81: 
82:   return (
83:     <Overlay
84:       isOpen={payload.isOpen}
85:       onClick={() => setWACodeModal({ ...defaultWACodeModal })}
86:     >
```

### `src/state/user/reducer.ts:8`
```text
3: import { TAction } from '../../types'
4: import { TRANSLATION } from '../../localization'
5: import { EStatuses } from '../../types/types'
6: 
7: export const record = Record<IUserState>({
8:   user: null,
9:   tokens: null,
10:   status: EStatuses.Default,
11:   message: '',
12:   tab: LOGIN_TABS_IDS[0],
13:   response: null,
14:   whatsappSignUpData: { u_phone: '' },
15: })
16: 
```

### `src/state/user/reducer.ts:25`
```text
20:   switch (type) {
21:     case ActionTypes.LOGIN_START:
22:       return state
23:         .set('status', EStatuses.Loading)
24:         .set('message', '')
25:         .set('user', null)
26:         .set('tokens', null)
27:     case ActionTypes.LOGIN_SUCCESS:
28:       return state
29:         .set('status', EStatuses.Success)
30:         .set('user', payload.user)
31:         .set('tokens', payload.tokens)
32:     case ActionTypes.LOGIN_FAIL:
33:       console.log('LOGIN_FAIL', payload)
```

### `src/state/user/reducer.ts:30`
```text
25:         .set('user', null)
26:         .set('tokens', null)
27:     case ActionTypes.LOGIN_SUCCESS:
28:       return state
29:         .set('status', EStatuses.Success)
30:         .set('user', payload.user)
31:         .set('tokens', payload.tokens)
32:     case ActionTypes.LOGIN_FAIL:
33:       console.log('LOGIN_FAIL', payload)
34:       return state
35:         .set('status', EStatuses.Fail)
36:         .set('message', payload)
37:     case ActionTypes.LOGIN_WHATSAPP:
38:       return state
```

### `src/state/user/reducer.ts:46`
```text
41: 
42:     case ActionTypes.GOOGLE_LOGIN_START:
43:       return state
44:         .set('status', EStatuses.Loading)
45:         .set('message', '')
46:         .set('user', null)
47:         .set('tokens', null)
48:     case ActionTypes.GOOGLE_LOGIN_SUCCESS:
49:       return state
50:         .set('status', EStatuses.Success)
51:         .set('user', payload.user)
52:         .set('tokens', payload.tokens)
53:     case ActionTypes.GOOGLE_LOGIN_FAIL:
54:       return state
```

### `src/state/user/reducer.ts:51`
```text
46:         .set('user', null)
47:         .set('tokens', null)
48:     case ActionTypes.GOOGLE_LOGIN_SUCCESS:
49:       return state
50:         .set('status', EStatuses.Success)
51:         .set('user', payload.user)
52:         .set('tokens', payload.tokens)
53:     case ActionTypes.GOOGLE_LOGIN_FAIL:
54:       return state
55:         .set('status', EStatuses.Fail)
56:         .set('message', TRANSLATION.LOGIN_FAIL)
57: 
58:     case ActionTypes.LOGOUT_START:
59:       return state
```

### `src/state/user/reducer.ts:62`
```text
57: 
58:     case ActionTypes.LOGOUT_START:
59:       return state
60:         .set('status', EStatuses.Loading)
61:         .set('message', '')
62:         .set('user', null)
63:         .set('tokens', null)
64:     case ActionTypes.LOGOUT_SUCCESS:
65:       return state
66:         .set('status', EStatuses.Success)
67:         .set('message', TRANSLATION.LOGOUT_SUCCESS)
68:     case ActionTypes.LOGOUT_FAIL:
69:       return state
70:         .set('status', EStatuses.Fail)
```

### `src/state/user/reducer.ts:77`
```text
72: 
73:     case ActionTypes.REGISTER_START:
74:       return state
75:         .set('status', EStatuses.Loading)
76:         .set('message', '')
77:         .set('user', null)
78:         .set('tokens', null)
79:     case ActionTypes.REGISTER_SUCCESS:
80:       return state
81:         .set('status', EStatuses.Success)
82:         .set('message', TRANSLATION.REGISTER_SUCCESS)
83:         .set('response', payload)
84:     case ActionTypes.REGISTER_FAIL:
85:       return state
```

### `src/state/user/reducer.ts:93`
```text
88: 
89:     case ActionTypes.REMIND_PASSWORD_START:
90:       return state
91:         .set('status', EStatuses.Loading)
92:         .set('message', '')
93:         .set('user', null)
94:         .set('tokens', null)
95:     case ActionTypes.REMIND_PASSWORD_SUCCESS:
96:       return state
97:         .set('status', EStatuses.Success)
98:         .set('message', TRANSLATION.REMIND_PASSWORD_SUCCESS)
99:     case ActionTypes.REMIND_PASSWORD_FAIL:
100:       return state
101:         .set('status', EStatuses.Fail)
```

### `src/state/user/reducer.ts:114`
```text
109:     case ActionTypes.SET_TOKENS:
110:       return state
111:         .set('tokens', payload)
112:     case ActionTypes.SET_USER:
113:       return state
114:         .set('user', payload)
115:         .set('tab', payload ? LOGIN_TABS_IDS[0] : LOGIN_TABS_IDS[1])
116: 
117:     case ActionTypes.WHATSAPP_SIGNUP_START:
118:       return state
119:         .set('status', EStatuses.Loading)
120:         .set('message', '')
121:         .set('whatsappSignUpData', { u_phone: payload })
122:     case ActionTypes.WHATSAPP_SIGNUP_SUCCESS:
```

### `src/state/user/selectors.ts:6`
```text
1: import { IRootState } from './../index'
2: import { createSelector } from 'reselect'
3: import { moduleName } from './constants'
4: 
5: export const moduleSelector = (state: IRootState) => state[moduleName]
6: export const user = createSelector(moduleSelector, state => state.user)
7: export const tokens = createSelector(moduleSelector, state => state.tokens)
8: export const status = createSelector(moduleSelector, state => state.status)
9: export const message = createSelector(moduleSelector, state => state.message)
10: export const tab = createSelector(moduleSelector, state => state.tab)
11: export const registerResponse = createSelector(moduleSelector, state => state.response)
12: export const whatsappSignUpData = createSelector(moduleSelector, state => state.whatsappSignUpData)
```

### `src/state/user/helpers.ts:26`
```text
21:   return Promise.all(uploads).then(res => {
22:     const u_details = res.reduce((acc, item) => ({ ...acc, ...item }), additionalDetails)
23:     return API.editUserAfterRegister({
24:       u_details,
25:       u_id: `${params?.u_id}`,
26:       token: params?.token,
27:       u_hash: params?.u_hash,
28:     })
29:   })
30: }
31: 
32: export function uploadRegisterFiles(params: any) {
33:   const { filesToUpload, response, u_details } = params
34:   const uploadsName: string[] = []
```

### `src/state/user/helpers.ts:27`
```text
22:     const u_details = res.reduce((acc, item) => ({ ...acc, ...item }), additionalDetails)
23:     return API.editUserAfterRegister({
24:       u_details,
25:       u_id: `${params?.u_id}`,
26:       token: params?.token,
27:       u_hash: params?.u_hash,
28:     })
29:   })
30: }
31: 
32: export function uploadRegisterFiles(params: any) {
33:   const { filesToUpload, response, u_details } = params
34:   const uploadsName: string[] = []
35:   const uploads = filesToUpload
```

### `src/state/user/helpers.ts:63`
```text
58:   })
59:     .then(u_details => {
60:       return API.editUserAfterRegister({
61:         u_details,
62:         u_id: `${response?.u_id}`,
63:         token: response?.token,
64:         u_hash: response?.u_hash,
65:       })
66:     })
67: }
```

### `src/state/user/helpers.ts:64`
```text
59:     .then(u_details => {
60:       return API.editUserAfterRegister({
61:         u_details,
62:         u_id: `${response?.u_id}`,
63:         token: response?.token,
64:         u_hash: response?.u_hash,
65:       })
66:     })
67: }
```

### `src/state/user/sagas.ts:35`
```text
30: }
31: 
32: function* loginSaga(data: TAction) {
33:   yield put({ type: ActionTypes.LOGIN_START })
34:   try {
35:     const result = yield* call(API.login, data.payload)
36:     if (!result) throw new Error('wrong login response')
37: 
38:     if(result.data === 'wrong login') {
39:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
40:       throw new Error('wrong login')
41:     }
42:     if(result.data === 'wrong password') {
43:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
```

### `src/state/user/sagas.ts:36`
```text
31: 
32: function* loginSaga(data: TAction) {
33:   yield put({ type: ActionTypes.LOGIN_START })
34:   try {
35:     const result = yield* call(API.login, data.payload)
36:     if (!result) throw new Error('wrong login response')
37: 
38:     if(result.data === 'wrong login') {
39:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
40:       throw new Error('wrong login')
41:     }
42:     if(result.data === 'wrong password') {
43:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
44:       throw new Error('wrong password')
```

### `src/state/user/sagas.ts:38`
```text
33:   yield put({ type: ActionTypes.LOGIN_START })
34:   try {
35:     const result = yield* call(API.login, data.payload)
36:     if (!result) throw new Error('wrong login response')
37: 
38:     if(result.data === 'wrong login') {
39:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
40:       throw new Error('wrong login')
41:     }
42:     if(result.data === 'wrong password') {
43:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
44:       throw new Error('wrong password')
45:     }
46: 
```

### `src/state/user/sagas.ts:40`
```text
35:     const result = yield* call(API.login, data.payload)
36:     if (!result) throw new Error('wrong login response')
37: 
38:     if(result.data === 'wrong login') {
39:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
40:       throw new Error('wrong login')
41:     }
42:     if(result.data === 'wrong password') {
43:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
44:       throw new Error('wrong password')
45:     }
46: 
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
```

### `src/state/user/sagas.ts:48`
```text
43:       yield put({ type: ActionTypes.LOGIN_FAIL, payload: result.data })
44:       throw new Error('wrong password')
45:     }
46: 
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
51:       return
52:     }
53: 
54:     if(result.data !== null && result.data === 'code sent') {
55:       yield put({ type: ActionTypes.LOGIN_WHATSAPP })
56:       return
```

### `src/state/user/sagas.ts:59`
```text
54:     if(result.data !== null && result.data === 'code sent') {
55:       yield put({ type: ActionTypes.LOGIN_WHATSAPP })
56:       return
57:     }
58: 
59:     if (result.user === null) throw new Error('wrong login response')
60: 
61:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
62: 
63: 
64:     if(result.user.u_role === EUserRoles.Client || result.user.u_role === EUserRoles.Agent) {
65:       data.payload.navigate('/passenger-order')
66:     } else if(result.user.u_role === EUserRoles.Driver) {
67:       data.payload.navigate('/driver-order')
```

### `src/state/user/sagas.ts:61`
```text
56:       return
57:     }
58: 
59:     if (result.user === null) throw new Error('wrong login response')
60: 
61:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
62: 
63: 
64:     if(result.user.u_role === EUserRoles.Client || result.user.u_role === EUserRoles.Agent) {
65:       data.payload.navigate('/passenger-order')
66:     } else if(result.user.u_role === EUserRoles.Driver) {
67:       data.payload.navigate('/driver-order')
68:     }
69: 
```

### `src/state/user/sagas.ts:64`
```text
59:     if (result.user === null) throw new Error('wrong login response')
60: 
61:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
62: 
63: 
64:     if(result.user.u_role === EUserRoles.Client || result.user.u_role === EUserRoles.Agent) {
65:       data.payload.navigate('/passenger-order')
66:     } else if(result.user.u_role === EUserRoles.Driver) {
67:       data.payload.navigate('/driver-order')
68:     }
69: 
70:     yield put({
71:       type: ActionTypes.LOGIN_SUCCESS,
72:       payload: result,
```

### `src/state/user/sagas.ts:66`
```text
61:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
62: 
63: 
64:     if(result.user.u_role === EUserRoles.Client || result.user.u_role === EUserRoles.Agent) {
65:       data.payload.navigate('/passenger-order')
66:     } else if(result.user.u_role === EUserRoles.Driver) {
67:       data.payload.navigate('/driver-order')
68:     }
69: 
70:     yield put({
71:       type: ActionTypes.LOGIN_SUCCESS,
72:       payload: result,
73:     })
74:     yield put(setLoginModal(false))
```

### `src/state/user/sagas.ts:90`
```text
85:   yield put({ type: ActionTypes.GOOGLE_LOGIN_START })
86:   try {
87: 
88:     const result = yield* call(API.googleLogin, data.payload)
89: 
90:     if (!result) throw new Error('Wrong login response')
91:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
92: 
93:     yield put({
94:       type: ActionTypes.GOOGLE_LOGIN_SUCCESS,
95:       payload: result,
96:     })
97:     yield put(setLoginModal(false))
98:     yield put(setRefCodeModal({ isOpen: false }))
```

### `src/state/user/sagas.ts:91`
```text
86:   try {
87: 
88:     const result = yield* call(API.googleLogin, data.payload)
89: 
90:     if (!result) throw new Error('Wrong login response')
91:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
92: 
93:     yield put({
94:       type: ActionTypes.GOOGLE_LOGIN_SUCCESS,
95:       payload: result,
96:     })
97:     yield put(setLoginModal(false))
98:     yield put(setRefCodeModal({ isOpen: false }))
99:   } catch (error) {
```

### `src/state/user/sagas.ts:117`
```text
112:       yield put({ type: ActionTypes.REGISTER_FAIL, payload: response.error })
113:       throw new Error(response.error)
114:     }
115: 
116:     const tokens = {
117:       token: response.token,
118:       u_hash: response.u_hash,
119:     }
120:     localStorage.setItem('state.user.tokens', JSON.stringify(tokens))
121: 
122:     if (data.payload?.u_role === 2) {
123:       if (uploads) {
124:         yield* call(uploadRegisterFiles, { filesToUpload: uploads, response, u_details })
125:       } else {
```

### `src/state/user/sagas.ts:118`
```text
113:       throw new Error(response.error)
114:     }
115: 
116:     const tokens = {
117:       token: response.token,
118:       u_hash: response.u_hash,
119:     }
120:     localStorage.setItem('state.user.tokens', JSON.stringify(tokens))
121: 
122:     if (data.payload?.u_role === 2) {
123:       if (uploads) {
124:         yield* call(uploadRegisterFiles, { filesToUpload: uploads, response, u_details })
125:       } else {
126:         const { passport_photo, driver_license_photo, license_photo, ...details } = u_details
```

### `src/state/user/sagas.ts:120`
```text
115: 
116:     const tokens = {
117:       token: response.token,
118:       u_hash: response.u_hash,
119:     }
120:     localStorage.setItem('state.user.tokens', JSON.stringify(tokens))
121: 
122:     if (data.payload?.u_role === 2) {
123:       if (uploads) {
124:         yield* call(uploadRegisterFiles, { filesToUpload: uploads, response, u_details })
125:       } else {
126:         const { passport_photo, driver_license_photo, license_photo, ...details } = u_details
127:         const files = { passport_photo, driver_license_photo, license_photo }
128:         const t = { ...tokens, u_id: response.u_id }
```

### `src/state/user/sagas.ts:161`
```text
156: }
157: 
158: function* logoutSaga() {
159:   yield put({ type: ActionTypes.LOGOUT_START })
160:   try {
161:     localStorage.removeItem('state.user.user')
162:     localStorage.removeItem('state.user.tokens')
163: 
164:     yield* call(API.logout)
165:     yield put({ type: ActionTypes.LOGOUT_SUCCESS })
166:     yield put(clearOrders())
167:   } catch (error) {
168:     console.error(error)
169:     yield put({ type: ActionTypes.LOGOUT_FAIL })
```

### `src/state/user/sagas.ts:162`
```text
157: 
158: function* logoutSaga() {
159:   yield put({ type: ActionTypes.LOGOUT_START })
160:   try {
161:     localStorage.removeItem('state.user.user')
162:     localStorage.removeItem('state.user.tokens')
163: 
164:     yield* call(API.logout)
165:     yield put({ type: ActionTypes.LOGOUT_SUCCESS })
166:     yield put(clearOrders())
167:   } catch (error) {
168:     console.error(error)
169:     yield put({ type: ActionTypes.LOGOUT_FAIL })
170:   }
```

### `src/state/user/sagas.ts:164`
```text
159:   yield put({ type: ActionTypes.LOGOUT_START })
160:   try {
161:     localStorage.removeItem('state.user.user')
162:     localStorage.removeItem('state.user.tokens')
163: 
164:     yield* call(API.logout)
165:     yield put({ type: ActionTypes.LOGOUT_SUCCESS })
166:     yield put(clearOrders())
167:   } catch (error) {
168:     console.error(error)
169:     yield put({ type: ActionTypes.LOGOUT_FAIL })
170:   }
171: }
172: 
```

### `src/state/user/sagas.ts:185`
```text
180:   }
181: }
182: 
183: function* initUserSaga() {
184:   try {
185:     const rawTokens = localStorage.getItem('state.user.tokens')
186:     const tokens: ITokens = rawTokens !== null ? JSON.parse(rawTokens) : {}
187:     if (!tokens.token || !tokens.u_hash) {
188:       // Проверяем язык в куках
189:       const savedLang = getCookie('user_lang')
190:       if (savedLang) {
191:         const language = SITE_CONSTANTS.LANGUAGES.find(i => i.iso === savedLang)
192:         if (language) {
193:           yield put({
```

### `src/state/user/sagas.ts:187`
```text
182: 
183: function* initUserSaga() {
184:   try {
185:     const rawTokens = localStorage.getItem('state.user.tokens')
186:     const tokens: ITokens = rawTokens !== null ? JSON.parse(rawTokens) : {}
187:     if (!tokens.token || !tokens.u_hash) {
188:       // Проверяем язык в куках
189:       const savedLang = getCookie('user_lang')
190:       if (savedLang) {
191:         const language = SITE_CONSTANTS.LANGUAGES.find(i => i.iso === savedLang)
192:         if (language) {
193:           yield put({
194:             type: ConfigActionTypes.SET_LANGUAGE,
195:             payload: language,
```

## 3. Evidence boundary

Для Semantic Graph authentication claim считается подтверждённым только если можно восстановить:

```text
frontend auth action
      ↓
credential/token state
      ↓
API request
      ↓
backend authentication mechanism```

Наличие строк `token`, `u_hash` или `id_role` само по себе не доказывает полный auth flow.

## 4. Preliminary semantic model

Исследование frontend показывает отдельные implementation fragments вокруг:

```text
User
Token
u_hash
Role
API request```

Но эти элементы не следует автоматически объединять в один `Authentication` claim без конкретного control/data flow.

## 5. Snapshot boundary

Все frontend claims данного документа принадлежат:

```text
Frontend = taxi-master.zip```

Их нельзя переносить на другой Taxi frontend или на другой versioned snapshot без нового evidence.

## 6. Gap Report

```text
G-31-01  identify concrete login/auth entry point        OPEN
G-31-02  trace token/u_hash persistence                  OPEN
G-31-03  trace API injection of auth credentials         OPEN
G-31-04  map frontend role to backend id_role              OPEN
G-31-05  trace logout / auth reset                        OPEN
```

## 7. Следующий шаг

Вместо широкого поиска взять конкретный auth API adapter и пройти его целиком:

```text
login action
   ↓
API request
   ↓
response
   ↓
token / u_hash
   ↓
frontend state/storage
   ↓
subsequent API request
```

После этого отдельно проверить:

```text
backend authenticated User        ↕frontend User / id_role```

Только после этой трассы добавлять Authentication relations в общий Semantic Graph.