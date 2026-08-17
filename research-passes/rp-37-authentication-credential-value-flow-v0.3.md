# Backend Semantic Graph — Research Pass 37
# Authentication Credential Value-Flow v0.3

**Статус:** EVIDENCE-GROUNDED / RECONCILIATION  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предыдущая версия:** RP-37 Authentication Credential Value-Flow v0.2  
**Источники:** `taxi-master.zip` + `archive_17012026_1259_clear.zip`

## 1. Research Question

> Какие конкретные authentication credentials автоматически передаёт общий frontend `apiMethod()`, и каким именно входом `check_auth_user()` Core Backend их проверяет?

Это последний узкий шаг перед возможным `CONFIRMED` для Frontend ↔ Core Backend Authentication.

## 2. Frontend `apiMethod()` evidence

Найдено contexts: **49**.

### `src/API/order.ts:17`
```text
5:   EBookingDriverState,
6:   EOrderTypes,
7:   EPaymentWays,
8:   IBookingAddresses,
9:   IBookingCoordinates,
10:   IOrder,
11:   IUser,
12: } from '../types/types'
13: import { IResponse } from '../types/api'
14: import { cloneFormData } from '../tools/utils'
15: import { convertOrder, reverseConvertOrder } from '../tools/convert'
16: import {
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
31:   b_driver_code: IOrder['b_driver_code']
32: }> {
33:   const defaults: Partial<IOrder> = {
34:     b_payment_way: EPaymentWays.Cash,
35:   }
36: 
37:   const converted = reverseConvertOrder({
38:     ...defaults,
39:     ...data,
```

### `src/API/order.ts:93`
```text
81:   result.b_id = result.b_id.toString()
82: 
83:   if (data.b_voting) {
84:     const params = cloneFormData(formData)
85:     addToFormData(params, {
86:       action: EBookingActions.SetConfirmState,
87:     })
88:     await axios.post(`${Config.API_URL}/drive/get/${result.b_id}`, params)
89:   }
90: 
91:   return result
92: }
93: export const postDrive = apiMethod<typeof _postDrive>(_postDrive)
94: 
95: const _cancelDrive = (
96:   { formData }: IApiMethodArguments,
97:   id: IOrder['b_id'],
98:   reason?: IOrder['b_cancel_reason'],
99: ) => {
100:   addToFormData(formData, {
101:     action: EBookingActions.SetCancelState,
102:     reason,
103:   })
104: 
105:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
106:     .then(res => res.data)
107: }
108: export const cancelDrive = apiMethod<typeof _cancelDrive>(_cancelDrive)
109: 
110: export const getOrders: <TType extends EOrderTypes>(
111:   type?: TType,
112:   filter?: TType extends EOrderTypes.Ready ? {
113:     carClasses?: boolean
114:     locationClasses?: boolean
115:   } : undefined
```

### `src/API/order.ts:108`
```text
96:   { formData }: IApiMethodArguments,
97:   id: IOrder['b_id'],
98:   reason?: IOrder['b_cancel_reason'],
99: ) => {
100:   addToFormData(formData, {
101:     action: EBookingActions.SetCancelState,
102:     reason,
103:   })
104: 
105:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
106:     .then(res => res.data)
107: }
108: export const cancelDrive = apiMethod<typeof _cancelDrive>(_cancelDrive)
109: 
110: export const getOrders: <TType extends EOrderTypes>(
111:   type?: TType,
112:   filter?: TType extends EOrderTypes.Ready ? {
113:     carClasses?: boolean
114:     locationClasses?: boolean
115:   } : undefined
116: ) => Promise<IResponse<'200', {
117:   booking: IOrder[]
118: }> | IResponse<'404', {
119:   detail?: 'used_car_not_found'
120: }>> = apiMethod(async(
121:   { formData }: IApiMethodArguments,
122:   type: EOrderTypes = EOrderTypes.Active,
123:   filter: {
124:     carClasses?: boolean
125:     locationClasses?: boolean
126:   } = {},
127: ) => {
128:   const userID = userSelectors.user(store.getState())?.u_id
129: 
130:   addToFormData(formData, {
```

### `src/API/order.ts:120`
```text
108: export const cancelDrive = apiMethod<typeof _cancelDrive>(_cancelDrive)
109: 
110: export const getOrders: <TType extends EOrderTypes>(
111:   type?: TType,
112:   filter?: TType extends EOrderTypes.Ready ? {
113:     carClasses?: boolean
114:     locationClasses?: boolean
115:   } : undefined
116: ) => Promise<IResponse<'200', {
117:   booking: IOrder[]
118: }> | IResponse<'404', {
119:   detail?: 'used_car_not_found'
120: }>> = apiMethod(async(
121:   { formData }: IApiMethodArguments,
122:   type: EOrderTypes = EOrderTypes.Active,
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
137:   const queryParams: string[] = []
138:   let URLAdditionalPath = ''
139:   switch (type) {
140:     case EOrderTypes.Active:
141:       queryParams.push('fields=00000000u1')
142:       break
```

### `src/API/order.ts:199`
```text
187:     },
188:   }
189: })
190: 
191: const _getOrder = (
192:   { formData }: IApiMethodArguments,
193:   id: IOrder['b_id'],
194: ): Promise<IOrder | null> => {
195:   return axios.post(`${Config.API_URL}/drive/get/${id}?fields=00000000u1`, formData)
196:     .then(res => res.data.data)
197:     .then(res => (res.booking && res.booking[id] && convertOrder(res.booking[id])) || null)
198: }
199: export const getOrder = apiMethod<typeof _getOrder>(_getOrder)
200: 
201: const _editOrder = (
202:   { formData }: IApiMethodArguments,
203:   id: IOrder['b_id'],
204:   data: IBookingAddresses & Stringify<IBookingCoordinates>,
205: ): Promise<IResponseFields> => {
206:   addToFormData(formData, {
207:     action: EBookingActions.Edit,
208:     data: JSON.stringify(data),
209:   })
210: 
211:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
212:     .then(res => res.data)
213: }
214: export const editOrder = apiMethod<typeof _editOrder>(_editOrder)
215: 
216: const _takeOrder = (
217:   { formData }: IApiMethodArguments,
218:   id: IOrder['b_id'],
219:   options: {
220:     votingNumber: number
221:     performers_price: number
```

### `src/API/order.ts:214`
```text
202:   { formData }: IApiMethodArguments,
203:   id: IOrder['b_id'],
204:   data: IBookingAddresses & Stringify<IBookingCoordinates>,
205: ): Promise<IResponseFields> => {
206:   addToFormData(formData, {
207:     action: EBookingActions.Edit,
208:     data: JSON.stringify(data),
209:   })
210: 
211:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
212:     .then(res => res.data)
213: }
214: export const editOrder = apiMethod<typeof _editOrder>(_editOrder)
215: 
216: const _takeOrder = (
217:   { formData }: IApiMethodArguments,
218:   id: IOrder['b_id'],
219:   options: {
220:     votingNumber: number
221:     performers_price: number
222:   },
223:   candidate?: boolean,
224: ): Promise<{
225:   /** Индекс водителя */
226:   c_index: string,
227:   /** Текущее число машин поездки с booking_driver_states=3,4,5,6 */
228:   current_cars_count: string,
229:   /** Необходимое число машин поездки */
230:   b_cars_count: string,
231:   /** Если изменился статус поезки */
232:   b_state?: '1->2' | null
233: }> => {
234:   const userID = userSelectors.user(store.getState())?.u_id
235:   if (!userID) {
236:     Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
```

### `src/API/order.ts:259`
```text
247:         data: JSON.stringify({
248:           c_id: car.c_id,
249:           c_payment_way: EPaymentWays.Cash,
250:           c_options: { performers_price: options.performers_price },
251:         }),
252:       })
253: 
254:       return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
255:         .then(res => res.data)
256:         .then(res => res.status === 'error' ? Promise.reject(res.message) : res)
257:     })
258: }
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
273:   })
274: 
275:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
276:     .then(res => res.data)
277:     .then(res => res.status === 'error' ? Promise.reject() : res)
278: }
279: export const chooseCandidate = apiMethod<typeof _chooseCandidate>(_chooseCandidate)
280: 
281: const _setOrderState = (
```

### `src/API/order.ts:279`
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
281: const _setOrderState = (
282:   { formData }: IApiMethodArguments,
283:   id: IOrder['b_id'],
284:   state: EBookingDriverState,
285: ) => {
286:   let action
287:   switch (state) {
288:     case EBookingDriverState.Arrived:
289:       action = EBookingActions.SetArriveState
290:       break
291:     case EBookingDriverState.Started:
292:       action = EBookingActions.SetStartState
293:       break
294:     case EBookingDriverState.Finished:
295:       action = EBookingActions.SetCompleteState
296:       break
297:     default:
298:       return Promise.reject()
299:   }
300: 
301:   addToFormData(formData, {
```

### `src/API/order.ts:309`
```text
297:     default:
298:       return Promise.reject()
299:   }
300: 
301:   addToFormData(formData, {
302:     action,
303:   })
304: 
305:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
306:     .then(res => res.data)
307:     .then(res => res.status === 'error' ? Promise.reject() : res)
308: }
309: export const setOrderState = apiMethod<typeof _setOrderState>(_setOrderState)
310: 
311: const _setOrderRating = (
312:   { formData }: IApiMethodArguments,
313:   id: IOrder['b_id'],
314:   value: number,
315: ) => {
316:   addToFormData(formData, {
317:     action: EBookingActions.SetRate,
318:     value,
319:   })
320: 
321:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
322:     .then(res => res.data)
323: }
324: /**
325:  * @param value rating from 1 till 5
326:  */
327: export const setOrderRating = apiMethod<typeof _setOrderRating>(_setOrderRating)
328: 
329: const _setWaitingTime = (
330:   { formData }: IApiMethodArguments,
331:   id: IOrder['b_id'],
```

### `src/API/order.ts:327`
```text
315: ) => {
316:   addToFormData(formData, {
317:     action: EBookingActions.SetRate,
318:     value,
319:   })
320: 
321:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
322:     .then(res => res.data)
323: }
324: /**
325:  * @param value rating from 1 till 5
326:  */
327: export const setOrderRating = apiMethod<typeof _setOrderRating>(_setOrderRating)
328: 
329: const _setWaitingTime = (
330:   { formData }: IApiMethodArguments,
331:   id: IOrder['b_id'],
332:   previous: number,
333:   additional: number = 180,
334: ) => {
335:   addToFormData(formData, {
336:     action: EBookingActions.SetWaitingTime,
337:     previous,
338:     additional,
339:   })
340: 
341:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
342:     .then(res => res.data)
343: }
344: /**
345:  * Adds time to wait
346:  * @param previous actual waiting time
347:  */
348: export const setWaitingTime = apiMethod<typeof _setWaitingTime>(_setWaitingTime)
```

### `src/API/order.ts:348`
```text
336:     action: EBookingActions.SetWaitingTime,
337:     previous,
338:     additional,
339:   })
340: 
341:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
342:     .then(res => res.data)
343: }
344: /**
345:  * Adds time to wait
346:  * @param previous actual waiting time
347:  */
348: export const setWaitingTime = apiMethod<typeof _setWaitingTime>(_setWaitingTime)
```

### `src/API/car.ts:5`
```text
1: import axios from 'axios'
2: import { EBookingLocationKinds, ICar, IUser } from '../types/types'
3: import { IResponse } from '../types/api'
4: import { convertCar } from '../tools/convert'
5: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
6: import Config from '../config'
7: import SITE_CONSTANTS from '../siteConstants'
8: 
9: type TCreateCar = Pick<ICar,
10:   'cm_id' |
11:   'seats' |
12:   'registration_plate' |
13:   'color' |
14:   'cc_id'
15: > & Partial<Pick<ICar,
16:   'photo' |
17:   'details'
18: >>
19: interface ICreateCarResponse {
20:   created_car: {
21:     c_id: ICar['c_id'],
22:     u_id: IUser['u_id']
23:   }
24: }
25: 
26: export const createCar = apiMethod(async(
27:   { formData }: IApiMethodArguments,
```

### `src/API/car.ts:26`
```text
14:   'cc_id'
15: > & Partial<Pick<ICar,
16:   'photo' |
17:   'details'
18: >>
19: interface ICreateCarResponse {
20:   created_car: {
21:     c_id: ICar['c_id'],
22:     u_id: IUser['u_id']
23:   }
24: }
25: 
26: export const createCar = apiMethod(async(
27:   { formData }: IApiMethodArguments,
28:   fields: {
29:     u_id: IUser['u_id']
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
44:     .then(res => res.data.data)
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
```

### `src/API/car.ts:47`
```text
35:     await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)
36:   return data
37: })
38: 
39: const _getCar = (
40:   { formData }: IApiMethodArguments,
41:   id: ICar['c_id'],
42: ): Promise<ICar | null> => {
43:   return axios.post(`${Config.API_URL}/car/${id}`, formData)
44:     .then(res => res.data.data)
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
```

### `src/API/car.ts:57`
```text
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
71: })
72: 
73: const setDefaultCarLicenses = apiMethod(async(
74:   { formData }: IApiMethodArguments,
75:   id: ICar['c_id'],
76:   country: keyof typeof SITE_CONSTANTS.COUNTRIES,
77: ): Promise<void> => {
78:   const locationClass = SITE_CONSTANTS.BOOKING_LOCATION_CLASSES
79:     .find(lc => lc.kind === EBookingLocationKinds.Intercity)!
```

### `src/API/car.ts:59`
```text
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
71: })
72: 
73: const setDefaultCarLicenses = apiMethod(async(
74:   { formData }: IApiMethodArguments,
75:   id: ICar['c_id'],
76:   country: keyof typeof SITE_CONSTANTS.COUNTRIES,
77: ): Promise<void> => {
78:   const locationClass = SITE_CONSTANTS.BOOKING_LOCATION_CLASSES
79:     .find(lc => lc.kind === EBookingLocationKinds.Intercity)!
80:   addToFormData(formData, { data: JSON.stringify({
81:     licenses: [{
```

### `src/API/car.ts:73`
```text
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
71: })
72: 
73: const setDefaultCarLicenses = apiMethod(async(
74:   { formData }: IApiMethodArguments,
75:   id: ICar['c_id'],
76:   country: keyof typeof SITE_CONSTANTS.COUNTRIES,
77: ): Promise<void> => {
78:   const locationClass = SITE_CONSTANTS.BOOKING_LOCATION_CLASSES
79:     .find(lc => lc.kind === EBookingLocationKinds.Intercity)!
80:   addToFormData(formData, { data: JSON.stringify({
81:     licenses: [{
82:       en: 'license',
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
```

### `src/API/car.ts:98`
```text
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

### `src/API/car.ts:112`
```text
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
121:     'color' |
122:     'photo' |
123:     'details' |
124:     'cc_id'
125:   >>,
126: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
127:   addToFormData(formData, { data: JSON.stringify(fields) })
128:   const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)
129:   return data
130: })
131: 
132: export const driveCar = apiMethod(async(
133:   { formData }: IApiMethodArguments,
134:   car: ICar,
```

### `src/API/car.ts:114`
```text
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
121:     'color' |
122:     'photo' |
123:     'details' |
124:     'cc_id'
125:   >>,
126: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
127:   addToFormData(formData, { data: JSON.stringify(fields) })
128:   const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)
129:   return data
130: })
131: 
132: export const driveCar = apiMethod(async(
133:   { formData }: IApiMethodArguments,
134:   car: ICar,
135: ): Promise<IResponse<'200', {}> | IResponse<'404', {
136:   detail?: 'not_modified'
```

### `src/API/car.ts:132`
```text
120:     'registration_plate' |
121:     'color' |
122:     'photo' |
123:     'details' |
124:     'cc_id'
125:   >>,
126: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
127:   addToFormData(formData, { data: JSON.stringify(fields) })
128:   const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)
129:   return data
130: })
131: 
132: export const driveCar = apiMethod(async(
133:   { formData }: IApiMethodArguments,
134:   car: ICar,
135: ): Promise<IResponse<'200', {}> | IResponse<'404', {
136:   detail?: 'not_modified'
137: }>> => {
138:   let { data: response } = await axios.post(
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
153:       syncUserResponse.code === '404' &&
154:       syncUserResponse.data.detail === 'not_modified'
```

### `src/API/car.ts:160`
```text
148:     response.code === '200' ||
149:     (response.code === '404' && response.data.detail === 'not_modified')
150:   ) {
151:     const syncUserResponse = await syncUserWithCar(car)
152:     if (!(syncUserResponse.code === '200' || (
153:       syncUserResponse.code === '404' &&
154:       syncUserResponse.data.detail === 'not_modified'
155:     ))) response = syncUserResponse
156:   }
157:   return response
158: })
159: 
160: const syncUserWithCar = apiMethod(async(
161:   { formData }: IApiMethodArguments,
162:   car: ICar,
163: ): Promise<IResponse<'200', {}> | IResponse<'404', {
164:   detail?: 'not_modified'
165: }>> => {
166:   const carClass = SITE_CONSTANTS.CAR_CLASSES[car.cc_id]
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
181:   { formData }: IApiMethodArguments,
182: ): Promise<ICar> {
```

### `src/API/car.ts:190`
```text
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

### `src/API/user.ts:4`
```text
1: import axios from 'axios'
2: import { IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
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
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
```

### `src/API/user.ts:15`
```text
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
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
```

### `src/API/user.ts:25`
```text
13:     .then(res => convertUser(res.user[id]) || null)
14: }
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
```

### `src/API/user.ts:109`
```text
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

### `src/API/auth.ts:4`
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

### `src/API/location.ts:3`
```text
1: import axios from 'axios'
2: import { IResponse } from '../types/api'
3: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
4: import Config from '../config'
5: 
6: export const sendPosition = apiMethod(async(
7:   { formData }: IApiMethodArguments,
8:   { latitude, longitude }: {
9:     latitude: number
10:     longitude: number
11:   },
12: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
13:   addToFormData(formData, { latitude, longitude })
14:   const { data } = await axios.post(`${Config.API_URL}/location`, formData)
15:   return data
16: })
```

### `src/API/location.ts:6`
```text
1: import axios from 'axios'
2: import { IResponse } from '../types/api'
3: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
4: import Config from '../config'
5: 
6: export const sendPosition = apiMethod(async(
7:   { formData }: IApiMethodArguments,
8:   { latitude, longitude }: {
9:     latitude: number
10:     longitude: number
11:   },
12: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
13:   addToFormData(formData, { latitude, longitude })
14:   const { data } = await axios.post(`${Config.API_URL}/location`, formData)
15:   return data
16: })
```

### `src/API/index.ts:18`
```text
6:   IAddressPoint,
7:   IBookingCoordinatesLatitude,
8:   IBookingCoordinatesLongitude,
9:   IOrder,
10:   IPlaceResponse,
11:   IRouteInfo,
12:   ISuggestion,
13:   ITrip,
14: } from '../types/types'
15: import { getBase64, getHints } from '../tools/utils'
16: import { convertTrip, reverseConvertTrip } from '../tools/convert'
17: import {
18:   addToFormData, apiMethod, IApiMethodArguments, IResponseFields,
19: } from '../tools/api'
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
34:   whatsappSignUp,
35:   googleLogin,
36:   logout,
37: } from './auth'
38: export {
39:   getUser,
40:   getUsers,
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
```

### `src/API/index.ts:131`
```text
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
142:     .then(res => res.data)
143:     .then(res =>
144:       res.data.trip || [],
145:     )
146:     .then(res => res.map((item: any) => convertTrip(item)))
147:     .then(res =>
148:       res.sort(
149:         (a: IOrder, b: IOrder) => a.b_start_datetime < b.b_start_datetime ? 1 : -1,
150:       ),
151:     )
152: }
153: export const getTrips = apiMethod<typeof _getTrips>(_getTrips)
```

### `src/API/index.ts:153`
```text
141:   return axios.post(`${Config.API_URL}/trip`, formData)
142:     .then(res => res.data)
143:     .then(res =>
144:       res.data.trip || [],
145:     )
146:     .then(res => res.map((item: any) => convertTrip(item)))
147:     .then(res =>
148:       res.sort(
149:         (a: IOrder, b: IOrder) => a.b_start_datetime < b.b_start_datetime ? 1 : -1,
150:       ),
151:     )
152: }
153: export const getTrips = apiMethod<typeof _getTrips>(_getTrips)
154: 
155: const _getWashTrips = (
156:   { formData }: IApiMethodArguments,
157:   type: EOrderTypes = EOrderTypes.Active,
158: ): Promise<IOrder[]> => {
159:   addToFormData(formData, {
160:     array_type: 'list',
161:   })
162: 
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
```

### `src/API/index.ts:187`
```text
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
201: const _setOutDrive = (
202:   { formData }: IApiMethodArguments,
203:   isFinished: boolean,
204:   addresses?: {
205:     fromAddress?: string,
206:     fromLatitude?: string,
207:     fromLongitude?: string,
208:     toAddress?: string,
209:     toLatitude?: string,
```

### `src/API/index.ts:199`
```text
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
201: const _setOutDrive = (
202:   { formData }: IApiMethodArguments,
203:   isFinished: boolean,
204:   addresses?: {
205:     fromAddress?: string,
206:     fromLatitude?: string,
207:     fromLongitude?: string,
208:     toAddress?: string,
209:     toLatitude?: string,
210:     toLongitude?: string,
211:   },
212:   passengers?: IOrder['b_passengers_count'],
213: ): Promise<IResponseFields> => {
214:   addToFormData(formData, {
215:     'data': JSON.stringify(
216:       isFinished ?
217:         {
218:           out_drive: '0',
219:         } :
220:         {
221:           out_drive: '1',
```

### `src/API/index.ts:236`
```text
224:           out_s_longitude: addresses?.fromLongitude?.toString(),
225:           out_address: addresses?.toAddress,
226:           out_latitude: addresses?.toLatitude?.toString(),
227:           out_longitude: addresses?.toLongitude?.toString(),
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
242:     details?: boolean
243:   } = {},
244: ): Promise<IPlaceResponse> => {
245:   const language = configSelectors.language(store.getState())
246: 
247:   return axios.get(
248:     'https://nominatim.openstreetmap.org/reverse',
249:     {
250:       params: {
251:         lat,
252:         lon: lng,
253:         addressdetails: +details,
254:         format: 'json',
255:         'accept-language': language.iso,
256:       },
257:     },
258:   )
```

### `src/tools/api.ts:15`
```text
3: import state from '../state'
4: 
5: export interface IApiMethodArguments {
6:   token: string,
7:   uHash: string,
8:   formData: FormData
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
```

## 3. Frontend credential evidence

Найдено contexts: **54**.

### `src/serviceWorker.js:104`
```text
92:           }
93:         }
94:       }
95:     })
96:     .catch(error => {
97:       console.error('Error during service worker registration:', error)
98:     })
99: }
100: 
101: function checkValidServiceWorker(swUrl, config) {
102:   // Check if the service worker can be found. If it can't reload the page.
103:   fetch(swUrl, {
104:     headers: { 'Service-Worker': 'script' },
105:   })
106:     .then(response => {
107:       // Ensure service worker exists, and that we really are getting a JS file.
108:       const contentType = response.headers.get('content-type')
109:       if (
110:         response.status === 404 ||
111:         (contentType != null && contentType.indexOf('javascript') === -1)
112:       ) {
113:         // No service worker found. Probably a different app. Reload the page.
114:         navigator.serviceWorker.ready.then(registration => {
115:           registration.unregister().then(() => {
116:             window.location.reload()
117:           })
118:         })
119:       } else {
120:         // Service worker found. Proceed as normal.
121:         registerValidSW(swUrl, config)
122:       }
123:     })
124:     .catch(() => {
125:       console.log(
126:         'No internet connection found. App is running in offline mode.',
```

### `src/serviceWorker.js:108`
```text
96:     .catch(error => {
97:       console.error('Error during service worker registration:', error)
98:     })
99: }
100: 
101: function checkValidServiceWorker(swUrl, config) {
102:   // Check if the service worker can be found. If it can't reload the page.
103:   fetch(swUrl, {
104:     headers: { 'Service-Worker': 'script' },
105:   })
106:     .then(response => {
107:       // Ensure service worker exists, and that we really are getting a JS file.
108:       const contentType = response.headers.get('content-type')
109:       if (
110:         response.status === 404 ||
111:         (contentType != null && contentType.indexOf('javascript') === -1)
112:       ) {
113:         // No service worker found. Probably a different app. Reload the page.
114:         navigator.serviceWorker.ready.then(registration => {
115:           registration.unregister().then(() => {
116:             window.location.reload()
117:           })
118:         })
119:       } else {
120:         // Service worker found. Proceed as normal.
121:         registerValidSW(swUrl, config)
122:       }
123:     })
124:     .catch(() => {
125:       console.log(
126:         'No internet connection found. App is running in offline mode.',
127:       )
128:     })
129: }
130: 
```

### `config/webpack.config.js:625`
```text
613:       // the HTML & assets that are part of the webpack build.
614:       isEnvProduction &&
615:         new WorkboxWebpackPlugin.GenerateSW({
616:           clientsClaim: true,
617:           exclude: [/\.map$/, /asset-manifest\.json$/],
618:           navigateFallback: paths.publicUrlOrPath + 'index.html',
619:           navigateFallbackDenylist: [
620:             // Exclude URLs starting with /_, as they're likely an API call
621:             new RegExp('^/_'),
622:             // Exclude any URLs whose last part seems to be a file extension
623:             // as they're likely a resource and not a SPA route.
624:             // URLs containing a "?" character won't be blacklisted as they're likely
625:             // a route with query params (e.g. auth callbacks).
626:             new RegExp('/[^/?]+\\.[^/]+$'),
627:           ],
628:         }),
629:       // TypeScript type checking
630:       useTypeScript &&
631:         new ForkTsCheckerWebpackPlugin({
632:           typescript: {
633:             configFile: paths.appTsConfig,
634:             diagnosticOptions: {
635:               syntactic: true,
636:             },
637:           },
638:           async: isEnvDevelopment,
639:           // The formatter is invoked directly in WebpackDevServerUtils during development
640:           formatter: isEnvProduction ? typescriptFormatter : undefined,
641:         }),
642:     ].filter(Boolean),
643:     // Turn off performance processing because we utilize
644:     // our own hints via the FileSizeReporter
645:     performance: false,
646:   }
647: }
```

### `lib/react-dev-utils/evalSourceMapMiddleware.js:31`
```text
19: }
20: 
21: /*
22:  * Middleware responsible for retrieving a generated source
23:  * Receives a webpack internal url: "webpack-internal:///<module-id>"
24:  * Returns a generated source: "<source-text><sourceMappingURL><sourceURL>"
25:  *
26:  * Based on EvalSourceMapDevToolModuleTemplatePlugin.js
27:  */
28: module.exports = function createEvalSourceMapMiddleware(server) {
29:   return function handleWebpackInternalMiddleware(req, res, next) {
30:     if (req.url.startsWith('/__get-internal-source')) {
31:       const fileName = req.query.fileName;
32:       const id = fileName.match(/webpack-internal:\/\/\/(.+)/)[1];
33:       if (!id || !server._stats) {
34:         next();
35:       }
36: 
37:       const source = getSourceById(server, id);
38:       const sourceMapURL = `//# sourceMappingURL=${base64SourceMap(source)}`;
39:       const sourceURL = `//# sourceURL=webpack-internal:///${module.id}`;
40:       res.end(`${source.source()}\n${sourceMapURL}\n${sourceURL}`);
41:     } else {
42:       next();
43:     }
44:   };
45: };
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:321`
```text
309:       o.hostname = '127.0.0.1';
310:     }
311:   } catch (_ignored) {
312:     o.hostname = '127.0.0.1';
313:   }
314:   return url.format(o);
315: }
316: 
317: // We need to provide a custom onError function for httpProxyMiddleware.
318: // It allows us to log custom error messages on the console.
319: function onProxyError(proxy) {
320:   return (err, req, res) => {
321:     const host = req.headers && req.headers.host;
322:     console.log(
323:       chalk.red('Proxy error:') +
324:         ' Could not proxy request ' +
325:         chalk.cyan(req.url) +
326:         ' from ' +
327:         chalk.cyan(host) +
328:         ' to ' +
329:         chalk.cyan(proxy) +
330:         '.'
331:     );
332:     console.log(
333:       'See https://nodejs.org/api/errors.html#errors_common_system_errors for more information (' +
334:         chalk.cyan(err.code) +
335:         ').'
336:     );
337:     console.log();
338: 
339:     // And immediately send the proper error response to the client.
340:     // Otherwise, the request will eventually timeout with ERR_EMPTY_RESPONSE on the client side.
341:     if (res.writeHead && !res.headersSent) {
342:       res.writeHead(500);
343:     }
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:426`
```text
414:       // So if `proxy` is specified as a string, we need to decide which fallback to use.
415:       // We use a heuristic: We want to proxy all the requests that are not meant
416:       // for static assets and as all the requests for static assets will be using
417:       // `GET` method, we can proxy all non-`GET` requests.
418:       // For `GET` requests, if request `accept`s text/html, we pick /index.html.
419:       // Modern browsers include text/html into `accept` header when navigating.
420:       // However API calls like `fetch()` won’t generally accept text/html.
421:       // If this heuristic doesn’t work well for you, use `src/setupProxy.js`.
422:       context: function(pathname, req) {
423:         return (
424:           req.method !== 'GET' ||
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
437:       },
438:       onError: onProxyError(target),
439:       secure: false,
440:       changeOrigin: true,
441:       ws: true,
442:       xfwd: true,
443:     },
444:   ];
445: }
446: 
447: function choosePort(host, defaultPort) {
448:   return detect(defaultPort, host).then(
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:427`
```text
415:       // We use a heuristic: We want to proxy all the requests that are not meant
416:       // for static assets and as all the requests for static assets will be using
417:       // `GET` method, we can proxy all non-`GET` requests.
418:       // For `GET` requests, if request `accept`s text/html, we pick /index.html.
419:       // Modern browsers include text/html into `accept` header when navigating.
420:       // However API calls like `fetch()` won’t generally accept text/html.
421:       // If this heuristic doesn’t work well for you, use `src/setupProxy.js`.
422:       context: function(pathname, req) {
423:         return (
424:           req.method !== 'GET' ||
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
437:       },
438:       onError: onProxyError(target),
439:       secure: false,
440:       changeOrigin: true,
441:       ws: true,
442:       xfwd: true,
443:     },
444:   ];
445: }
446: 
447: function choosePort(host, defaultPort) {
448:   return detect(defaultPort, host).then(
449:     port =>
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:431`
```text
419:       // Modern browsers include text/html into `accept` header when navigating.
420:       // However API calls like `fetch()` won’t generally accept text/html.
421:       // If this heuristic doesn’t work well for you, use `src/setupProxy.js`.
422:       context: function(pathname, req) {
423:         return (
424:           req.method !== 'GET' ||
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
437:       },
438:       onError: onProxyError(target),
439:       secure: false,
440:       changeOrigin: true,
441:       ws: true,
442:       xfwd: true,
443:     },
444:   ];
445: }
446: 
447: function choosePort(host, defaultPort) {
448:   return detect(defaultPort, host).then(
449:     port =>
450:       new Promise(resolve => {
451:         if (port === defaultPort) {
452:           return resolve(port);
453:         }
```

### `lib/react-dev-utils/formatWebpackMessages.js:33`
```text
21:   if (typeof message === 'string') {
22:     lines = message.split('\n');
23:   } else if ('message' in message) {
24:     lines = message['message'].split('\n');
25:   } else if (Array.isArray(message)) {
26:     message.forEach(message => {
27:       if ('message' in message) {
28:         lines = message['message'].split('\n');
29:       }
30:     });
31:   }
32: 
33:   // Strip webpack-added headers off errors/warnings
34:   // https://github.com/webpack/webpack/blob/master/lib/ModuleError.js
35:   lines = lines.filter(line => !/Module [A-z ]+\(from/.test(line));
36: 
37:   // Transform parsing error into syntax error
38:   // TODO: move this to our ESLint formatter?
39:   lines = lines.map(line => {
40:     const parsingError = /Line (\d+):(?:(\d+):)?\s*Parsing error: (.+)$/.exec(
41:       line
42:     );
43:     if (!parsingError) {
44:       return line;
45:     }
46:     const [, errorLine, errorColumn, errorMessage] = parsingError;
47:     return `${friendlySyntaxErrorLabel} ${errorMessage} (${errorLine}:${errorColumn})`;
48:   });
49: 
50:   message = lines.join('\n');
51:   // Smoosh syntax errors (commonly found in CSS)
52:   message = message.replace(
53:     /SyntaxError\s+\((\d+):(\d+)\)\s*(.+?)\n/g,
54:     `${friendlySyntaxErrorLabel} $3 ($1:$2)\n`
55:   );
```

### `lib/react-dev-utils/errorOverlayMiddleware.js:15`
```text
3:  *
4:  * This source code is licensed under the MIT license found in the
5:  * LICENSE file in the root directory of this source tree.
6:  */
7: 'use strict';
8: 
9: const launchEditor = require('./launchEditor');
10: const launchEditorEndpoint = require('./launchEditorEndpoint');
11: 
12: module.exports = function createLaunchEditorMiddleware() {
13:   return function launchEditorMiddleware(req, res, next) {
14:     if (req.url.startsWith(launchEditorEndpoint)) {
15:       const lineNumber = parseInt(req.query.lineNumber, 10) || 1;
16:       const colNumber = parseInt(req.query.colNumber, 10) || 1;
17:       launchEditor(req.query.fileName, lineNumber, colNumber);
18:       res.end();
19:     } else {
20:       next();
21:     }
22:   };
23: };
```

### `lib/react-dev-utils/errorOverlayMiddleware.js:16`
```text
4:  * This source code is licensed under the MIT license found in the
5:  * LICENSE file in the root directory of this source tree.
6:  */
7: 'use strict';
8: 
9: const launchEditor = require('./launchEditor');
10: const launchEditorEndpoint = require('./launchEditorEndpoint');
11: 
12: module.exports = function createLaunchEditorMiddleware() {
13:   return function launchEditorMiddleware(req, res, next) {
14:     if (req.url.startsWith(launchEditorEndpoint)) {
15:       const lineNumber = parseInt(req.query.lineNumber, 10) || 1;
16:       const colNumber = parseInt(req.query.colNumber, 10) || 1;
17:       launchEditor(req.query.fileName, lineNumber, colNumber);
18:       res.end();
19:     } else {
20:       next();
21:     }
22:   };
23: };
```

### `lib/react-dev-utils/errorOverlayMiddleware.js:17`
```text
5:  * LICENSE file in the root directory of this source tree.
6:  */
7: 'use strict';
8: 
9: const launchEditor = require('./launchEditor');
10: const launchEditorEndpoint = require('./launchEditorEndpoint');
11: 
12: module.exports = function createLaunchEditorMiddleware() {
13:   return function launchEditorMiddleware(req, res, next) {
14:     if (req.url.startsWith(launchEditorEndpoint)) {
15:       const lineNumber = parseInt(req.query.lineNumber, 10) || 1;
16:       const colNumber = parseInt(req.query.colNumber, 10) || 1;
17:       launchEditor(req.query.fileName, lineNumber, colNumber);
18:       res.end();
19:     } else {
20:       next();
21:     }
22:   };
23: };
```

### `src/types/types.ts:621`
```text
609:     driver_license_photo?: number[]
610:     license_photo?: number[],
611:     subscribe?: boolean,
612:     carMark?: string,
613:     carModel?: string,
614:   }
615:   //
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
638: export interface IAddressPoint {
639:   address?: string,
640:   date?: string,
641:   time?: string
642:   shortAddress?: string,
643:   latitude?: number,
```

### `src/types/types.ts:622`
```text
610:     license_photo?: number[],
611:     subscribe?: boolean,
612:     carMark?: string,
613:     carModel?: string,
614:   }
615:   //
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
638: export interface IAddressPoint {
639:   address?: string,
640:   date?: string,
641:   time?: string
642:   shortAddress?: string,
643:   latitude?: number,
644:   longitude?: number
```

### `src/types/types.ts:627`
```text
615:   //
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
638: export interface IAddressPoint {
639:   address?: string,
640:   date?: string,
641:   time?: string
642:   shortAddress?: string,
643:   latitude?: number,
644:   longitude?: number
645: }
646: 
647: export interface ILoadedAddressPoint extends IAddressPoint {
648:   address: string
649:   latitude: number
```

### `src/types/types.ts:628`
```text
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
638: export interface IAddressPoint {
639:   address?: string,
640:   date?: string,
641:   time?: string
642:   shortAddress?: string,
643:   latitude?: number,
644:   longitude?: number
645: }
646: 
647: export interface ILoadedAddressPoint extends IAddressPoint {
648:   address: string
649:   latitude: number
650:   longitude: number
```

### `src/types/types.ts:753`
```text
741: export type TElevatorState = { [id: string]: number }
742: 
743: export interface IElevatorState {
744:   elevator?: boolean
745:   steps: TElevatorState
746: }
747: 
748: export type TBlockObject = { [key: string]: any }
749: 
750: export interface IRegisterResponse {
751:   u_id: number,
752:   string: string,
753:   token?: string,
754:   u_hash?: string
755: }
756: 
757: export type IFileUpload = {
758:   base64: string,
759:   name?: string,
760:   u_id?: string,
761:   private?: 0 | 1
762: }
763: 
764: 
765: export type TFilesMap = {
766:   passport_photo: any
767:   driver_license_photo: any
768:   license_photo: any
769: }
770: 
771: export interface IRequiredFields {
772:   [key: string]: boolean
773: }
774: 
775: export interface IProfitEstimationFactors {
```

### `src/types/types.ts:754`
```text
742: 
743: export interface IElevatorState {
744:   elevator?: boolean
745:   steps: TElevatorState
746: }
747: 
748: export type TBlockObject = { [key: string]: any }
749: 
750: export interface IRegisterResponse {
751:   u_id: number,
752:   string: string,
753:   token?: string,
754:   u_hash?: string
755: }
756: 
757: export type IFileUpload = {
758:   base64: string,
759:   name?: string,
760:   u_id?: string,
761:   private?: 0 | 1
762: }
763: 
764: 
765: export type TFilesMap = {
766:   passport_photo: any
767:   driver_license_photo: any
768:   license_photo: any
769: }
770: 
771: export interface IRequiredFields {
772:   [key: string]: boolean
773: }
774: 
775: export interface IProfitEstimationFactors {
776:   fuel_cost: number
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
```

### `src/API/index.ts:263`
```text
251:         lat,
252:         lon: lng,
253:         addressdetails: +details,
254:         format: 'json',
255:         'accept-language': language.iso,
256:       },
257:     },
258:   )
259:     .then(res => res.data)
260: }
261: 
262: export const geocode = (
263:   query: string,
264:   { details = false }: {
265:     details?: boolean
266:   } = {},
267: ): Promise<IPlaceResponse | null> => {
268:   const language = configSelectors.language(store.getState())
269: 
270:   return axios.get(
271:     'https://nominatim.openstreetmap.org/search',
272:     {
273:       params: {
274:         q: query,
275:         addressdetails: +details,
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
```

### `src/API/index.ts:274`
```text
262: export const geocode = (
263:   query: string,
264:   { details = false }: {
265:     details?: boolean
266:   } = {},
267: ): Promise<IPlaceResponse | null> => {
268:   const language = configSelectors.language(store.getState())
269: 
270:   return axios.get(
271:     'https://nominatim.openstreetmap.org/search',
272:     {
273:       params: {
274:         q: query,
275:         addressdetails: +details,
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
```

### `src/localization/common.ts:450`
```text
438:   TAKE_ORDER: 'take_order',
439:   TALL_CHEST: 'tall_chest',
440:   TALL_LAMP: 'tall_lamp',
441:   TEXT_ON_THE_TABLE: 'text_on_the_table',
442:   TIE_CARD_ARTICLE: 'tie_card_article',
443:   TIE_CARD_DESCRIPTION: 'tie_card_description',
444:   TIE_CARD_HEADER: 'tie_card_header',
445:   TIMER: 'timer',
446:   TIME_FROM: 'time_from',
447:   TIME_TILL: 'time_till',
448:   TO: 'to',
449:   TODAY: 'today',
450:   TOKEN: 'token',
451:   TOMORROW: 'tomorrow',
452:   TOOK_PASSENGER: 'took_passenger',
453:   TOOL_BOX: 'tool_box',
454:   TOP_P: 'top_p',
455:   TOY_STORAGE: 'toy_storage',
456:   TO_ORDER: 'to_order',
457:   TRAMPOLINE: 'trampoline',
458:   TRANSFER_DATE: 'transfer_date',
459:   TRANSPORTATION_DATE: 'transportation_date',
460:   TRANSPORT_P: 'transport_p',
461:   TRASH_CAN: 'trash_can',
462:   TREADMILL: 'treadmill',
463:   TRIP: 'trip',
464:   TRIPS_P: 'trips_p',
465:   TRUCK: 'truck',
466:   TV: 'tv',
467:   TV_OVER_75: 'tv_over_75',
468:   TV_STAND: 'tv_stand',
469:   TV_STORAGE_GLASS: 'tv_storage_glass',
470:   TV_TABLE: 'tv_table',
471:   TV_UNDER_75: 'tv_under_75',
472:   TWIN_BED: 'twin_bed',
```

### `src/localization/common.ts:482`
```text
470:   TV_TABLE: 'tv_table',
471:   TV_UNDER_75: 'tv_under_75',
472:   TWIN_BED: 'twin_bed',
473:   TWIN_BOX_SPRING: 'twin_box_spring',
474:   TWIN_MATTRESS: 'twin_mattress',
475:   TYPE_RELOCATION: 'type_relocation',
476:   UMBRELLA: 'umbrella',
477:   UP_TO_100: 'up_to_100',
478:   URBAN: 'urban',
479:   URBAN_P: 'urban_p',
480:   URGENTLY: 'urgently',
481:   USER_IS_NOT_PERFORMER_ERROR: 'user_is_not_performer_error',
482:   U_HASH: 'u_hash',
483:   UNAUTHORIZED_ACCESS: 'unauthorized_access',
484:   VERY_EXPENSIVE: 'very_expensive',
485:   VIEW_P: 'view_p',
486:   VOTE: 'vote',
487:   VOTER: 'voter',
488:   VOTING: 'voting',
489:   WAGON: 'wagon',
490:   WAITING: 'waiting',
491:   WAITING_FOR_LONG: 'waiting_for_long',
492:   WARDROBE: 'wardrobe',
493:   WARDROBE_BOX: 'wardrobe_box',
494:   WASHER: 'washer',
495:   WAY: 'way',
496:   WENT: 'went',
497:   WHAT_PICK_UP: 'what_pick_up',
498:   WHAT_WE_DELIVERING: 'what_we_delivering',
499:   WHEN: 'when',
500:   WHERE_FROM_P: 'where_from_p',
501:   WHERE_P: 'where_p',
502:   WORKOUT_BENCH: 'workout_bench',
503:   WRITE_COMMENT: 'write_comment',
504:   WRITE_COURIER_MISSION: 'write_courier_mission',
```

### `src/tools/api.ts:6`
```text
1: import { userSelectors } from '../state/user'
2: import { ParametersExceptFirst } from '../types'
3: import state from '../state'
4: 
5: export interface IApiMethodArguments {
6:   token: string,
7:   uHash: string,
8:   formData: FormData
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
```

### `src/tools/api.ts:32`
```text
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
49:   affected_fields: string[],
50:   /** Список невалидных ключей */
51:   forbidden_fields: string[],
52:   /** Список ключей с некорректными данными */
53:   wrong_data_fields?: string[]
54: }
```

### `src/tools/api.ts:33`
```text
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
49:   affected_fields: string[],
50:   /** Список невалидных ключей */
51:   forbidden_fields: string[],
52:   /** Список ключей с некорректными данными */
53:   wrong_data_fields?: string[]
54: }
55: 
```

### `src/tools/api.ts:38`
```text
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
49:   affected_fields: string[],
50:   /** Список невалидных ключей */
51:   forbidden_fields: string[],
52:   /** Список ключей с некорректными данными */
53:   wrong_data_fields?: string[]
54: }
55: 
56: export const addToFormData = (formData: FormData, object: {[key: string]: any}) => {
57:   for (let [key, value] of Object.entries(object)) {
58:     if (!value) continue
59:     formData.append(key, value)
60:   }
```

### `src/tools/api.ts:39`
```text
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
49:   affected_fields: string[],
50:   /** Список невалидных ключей */
51:   forbidden_fields: string[],
52:   /** Список ключей с некорректными данными */
53:   wrong_data_fields?: string[]
54: }
55: 
56: export const addToFormData = (formData: FormData, object: {[key: string]: any}) => {
57:   for (let [key, value] of Object.entries(object)) {
58:     if (!value) continue
59:     formData.append(key, value)
60:   }
61:   return formData
```

### `src/components/modals/ProfileModal.tsx:277`
```text
265:         if (!imagesMap[key]) imagesMap[key] = []
266:         return Promise.all(
267:           imageList
268:             .map((image: [any, File]) => {
269:               if (image[0]) imagesMap[key].push(image[0])
270:               return image
271:             })
272:             .filter((image: [any, File]) => !image[0])
273:             .map((image: [any, File]) =>
274:               API.uploadFile({
275:                 file: image[1],
276:                 u_id: user!.u_id,
277:                 token: tokens?.token,
278:                 u_hash: tokens?.u_hash,
279:               }).then(res => {
280:                 if (res?.dl_id) imagesMap[key].push(res.dl_id)
281:               }),
282:             ),
283:         )
284:       }))
285: 
286:       const fields = user!.u_check_state === EUserCheckStates.Required ||
287:         !user!.u_check_state ?
288:         DRIVER_CHECK_REQUIRED_FIELDS :
289:         user!.u_check_state === EUserCheckStates.Active ?
290:           DRIVER_CHECK_ACTIVE_FIELDS :
291:           new Set()
292:       try {
293:         await API.editUser({
294:           ...Object.fromEntries(Object.entries(values)
295:             .filter(([key]) => fields.has(key as any)),
296:           ) as any,
297:           u_details: { ...u_details, ...imagesMap },
298:         })
299:         updateUser()
```

### `src/components/modals/ProfileModal.tsx:278`
```text
266:         return Promise.all(
267:           imageList
268:             .map((image: [any, File]) => {
269:               if (image[0]) imagesMap[key].push(image[0])
270:               return image
271:             })
272:             .filter((image: [any, File]) => !image[0])
273:             .map((image: [any, File]) =>
274:               API.uploadFile({
275:                 file: image[1],
276:                 u_id: user!.u_id,
277:                 token: tokens?.token,
278:                 u_hash: tokens?.u_hash,
279:               }).then(res => {
280:                 if (res?.dl_id) imagesMap[key].push(res.dl_id)
281:               }),
282:             ),
283:         )
284:       }))
285: 
286:       const fields = user!.u_check_state === EUserCheckStates.Required ||
287:         !user!.u_check_state ?
288:         DRIVER_CHECK_REQUIRED_FIELDS :
289:         user!.u_check_state === EUserCheckStates.Active ?
290:           DRIVER_CHECK_ACTIVE_FIELDS :
291:           new Set()
292:       try {
293:         await API.editUser({
294:           ...Object.fromEntries(Object.entries(values)
295:             .filter(([key]) => fields.has(key as any)),
296:           ) as any,
297:           u_details: { ...u_details, ...imagesMap },
298:         })
299:         updateUser()
300:         setMessageModal({
```

### `src/components/modals/LoginModal/Register.tsx:195`
```text
183:     if (status === EStatuses.Fail && !isRegistrationAlertVisible) {
184:       toggleRegistrationAlertVisibility()
185:     }
186: 
187:     // if (status === EStatuses.Success && type === ERegistrationType.Phone && shouldSendToWhatsapp) {
188:     //   if (response) {
189:     //     axios.post(`${WHATSAPP_BOT_URL}/send-message`,
190:     //       {
191:     //         phone: u_phone,
192:     //         code: response.string,
193:     //       },
194:     //       {
195:     //         headers: {
196:     //           'x-api-key': WHATSAPP_BOT_KEY,
197:     //         },
198:     //       },
199:     //     ).then((response) => {
200:     //       console.log(response)
201:     //       whatsappResponseMessage = response.data
202:     //     }).catch((err) => {
203:     //       console.log(err)
204:     //       whatsappResponseMessage = err
205:     //     }).finally(() => {
206:     //       toggleRegistrationAlertVisibility()
207:     //       toggleWhatsappAlertVisibility()
208:     //     })
209:     //   }
210:     // }
211:   }, [status])
212: 
213:   useEffect(() => {
214:     let newData = (window as any).data
215: 
216:     if (newData && (data === null || data === undefined)) {
217:       setData({
```

### `src/state/user/helpers.ts:26`
```text
14:       return Promise.all(promises).then(
15:         (res: any[]) => res.reduce((acc, item) => ({
16:           [key]: [...acc[key], item.data?.data?.dl_id],
17:         }), { [key]: [] }),
18:       ).then((res: any) => ({ [key]: JSON.stringify(res[key]) }))
19:     })
20: 
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
35:   const uploads = filesToUpload
36:     .filter((item: any) => item.file)
37:     .map((item: any) => {
38:       uploadsName.push(item.name)
39:       return API.uploadFile({ file: item.file, ...response })
40:     })
41:   return Promise.all(uploads).then((res: any[]) => {
42:     const userData: Record<string, any> = {}
43:     res.forEach((item, i) => {
44:       const resData = item.data || {}
45:       if (resData.status !== 'success') return
46:       const fileId = resData.data?.dl_id
47:       userData[uploadsName[i]] = (userData[uploadsName[i]] || []).concat(fileId)
48:     })
```

### `src/state/user/helpers.ts:27`
```text
15:         (res: any[]) => res.reduce((acc, item) => ({
16:           [key]: [...acc[key], item.data?.data?.dl_id],
17:         }), { [key]: [] }),
18:       ).then((res: any) => ({ [key]: JSON.stringify(res[key]) }))
19:     })
20: 
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
35:   const uploads = filesToUpload
36:     .filter((item: any) => item.file)
37:     .map((item: any) => {
38:       uploadsName.push(item.name)
39:       return API.uploadFile({ file: item.file, ...response })
40:     })
41:   return Promise.all(uploads).then((res: any[]) => {
42:     const userData: Record<string, any> = {}
43:     res.forEach((item, i) => {
44:       const resData = item.data || {}
45:       if (resData.status !== 'success') return
46:       const fileId = resData.data?.dl_id
47:       userData[uploadsName[i]] = (userData[uploadsName[i]] || []).concat(fileId)
48:     })
49:     Object.keys(userData).forEach(key => {
```

### `src/state/user/helpers.ts:63`
```text
51:     })
52:     if (u_details) {
53:       Object.keys(u_details).forEach(key => {
54:         userData[key] = u_details[key]
55:       })
56:     }
57:     return userData
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
52:     if (u_details) {
53:       Object.keys(u_details).forEach(key => {
54:         userData[key] = u_details[key]
55:       })
56:     }
57:     return userData
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

### `src/state/user/sagas.ts:117`
```text
105: function* registerSaga(data: TAction) {
106:   yield put({ type: ActionTypes.REGISTER_START })
107:   try {
108:     const { uploads, u_details, u_car, ...payload } = data.payload
109: 
110:     const response: any = yield* call(API.register, { ...payload, st: 1 })
111:     if (response.error) {
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
126:         const { passport_photo, driver_license_photo, license_photo, ...details } = u_details
127:         const files = { passport_photo, driver_license_photo, license_photo }
128:         const t = { ...tokens, u_id: response.u_id }
129:         yield* call(uploadFiles, { files, u_details: details, tokens: t })
130:       }
131:     }
132:     yield put({ type: ActionTypes.REGISTER_SUCCESS, payload: response })
133:     yield* call(initUserSaga)
134:     yield put(setLoginModal(false))
135: 
136:     const carResponse = u_car && payload.u_role === EUserRoles.Driver ?
137:       (yield* putResolve(createUserCar({
138:         ...u_car,
139:         country: SITE_CONSTANTS.CITIES[payload.u_city ?? u_details.city]
```

### `src/state/user/sagas.ts:118`
```text
106:   yield put({ type: ActionTypes.REGISTER_START })
107:   try {
108:     const { uploads, u_details, u_car, ...payload } = data.payload
109: 
110:     const response: any = yield* call(API.register, { ...payload, st: 1 })
111:     if (response.error) {
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
126:         const { passport_photo, driver_license_photo, license_photo, ...details } = u_details
127:         const files = { passport_photo, driver_license_photo, license_photo }
128:         const t = { ...tokens, u_id: response.u_id }
129:         yield* call(uploadFiles, { files, u_details: details, tokens: t })
130:       }
131:     }
132:     yield put({ type: ActionTypes.REGISTER_SUCCESS, payload: response })
133:     yield* call(initUserSaga)
134:     yield put(setLoginModal(false))
135: 
136:     const carResponse = u_car && payload.u_role === EUserRoles.Driver ?
137:       (yield* putResolve(createUserCar({
138:         ...u_car,
139:         country: SITE_CONSTANTS.CITIES[payload.u_city ?? u_details.city]
140:           ?.country,
```

### `src/state/user/sagas.ts:187`
```text
175:   try {
176:     yield* call(API.remindPassword, data.payload)
177:     yield put({ type: ActionTypes.REMIND_PASSWORD_SUCCESS })
178:   } catch (error) {
179:     yield put({ type: ActionTypes.REMIND_PASSWORD_FAIL })
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
194:             type: ConfigActionTypes.SET_LANGUAGE,
195:             payload: language,
196:           })
197:         }
198:       }
199:       return
200:     }
201:     yield put({ type: ActionTypes.SET_TOKENS, payload: tokens })
202: 
203:     const user = yield* call(API.getAuthorizedUser)
204:     if (!user) {
205:       localStorage.removeItem('state.user.tokens')
206:       return
207:     }
208: 
209:     // Устанавливаем язык из пользователя или из куки
```

## 4. Core Backend `check_auth_user()` definition

Найдено contexts: **1**.

### `archive_17012026_1259/taxi/models/m_functions.php:247`
```php
235: 				`id_user` ='" . $id_user . "' AND (`expire_datetime` = 0 OR NOW() < `expire_datetime`)
236: 			";
237: 
238: 		$q = query($s);
239: 		@$d = fetch_assoc($q);
240: 		return array(
241: 			  'auth'		=> $d !== NULL ? $d['a'] : true,
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
256: 				`middle`,
257: 				`phone`,
258: 				`email`,
259: 				`photo_link`,
260: 				`id_lang`,
261: 				`currency`,
262: 				`id_navigation`,
263: 				`id_verification_status`,
264: 				`deleted`,
265: 				`active`,
266: 				`birthday_date`,
267: 				`tg`,
268: 				`wa`
269: 			FROM `users`
```

## 5. Core Backend credential extraction evidence

Найдено contexts: **138**.

### `archive_17012026_1259/taxi/index.php:6`
```php
1: <?php
2: 	header("Content-Type: text/html; charset=utf-8");
3: 	require_once('config/config.php');
4: 	if (ROOT_URL != '/') ini_set('session.cookie_path', ROOT_URL);
5: 	if (UID_SUFFIX != '') session_name(session_name() . UID_SUFFIX);
6: 	require_once('models/token.php');
7: 	if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
8: 	{
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
23: 		else
24: 		{
25: 			$AUID = get_id_user(isset($_POST['u_a_id'])?trim($_POST['u_a_id']):"",isset($_POST['u_a_email'])?trim($_POST['u_a_email']):"",isset($_POST['u_a_phone'])?trim($_POST['u_a_phone']):"",isset($_POST['u_a_tg'])?trim($_POST['u_a_tg']):"",isset($_POST['u_a_wa'])?trim($_POST['u_a_wa']):"");
26: 			if (is_array($AUID))
27: 			{
28: 				show_error($AUID);
```

### `archive_17012026_1259/taxi/models/api.php:318`
```php
306: 
307: 			$s = "DELETE
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
339: 		{
340: 			if (!empty($_SESSION[UID])) {
```

### `archive_17012026_1259/taxi/models/api.php:319`
```php
307: 			$s = "DELETE
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
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
```

### `archive_17012026_1259/taxi/models/api.php:321`
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
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
```

### `archive_17012026_1259/taxi/models/api.php:325`
```php
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
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
```

### `archive_17012026_1259/taxi/models/api.php:326`
```php
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
342: 			}
343: 
344: 			if (empty($login)) 
345: 			{
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
```

### `archive_17012026_1259/taxi/models/api.php:327`
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
346: 				return $this->showError('404', 'error', 'empty login');
347: 			}
348: 			if (in_array($type,$this->auth_type_arr))
349: 			{
```

### `archive_17012026_1259/taxi/models/api.php:8341`
```php
8329: 			elseif (mysqli_affected_rows($link) === 0) 
8330: 			{
8331: 				return $this->showError('404', 'error', 'modified data not found');
8332: 			}
8333: 			return array(
8334: 				'code' 		=>	'200',
8335: 				'status' 	=>	'success'
8336: 			);
8337: 		}
8338: 
8339: 		public $associativeArray = true;
8340: 
8341: 		public function selectToken($id_user = "", $token = array())
8342: 		{
8343: 			if (empty($_SESSION[UID])) {
8344: 				if (!empty($_POST['auth_hash']))
8345: 				{
8346: 					list($_COOKIE[session_name()],$_COOKIE['vfoc']) = array_merge(explode('|', openssl____decrypt($_POST['auth_hash'])),array(''));
8347: 					if (!empty($_COOKIE[session_name()])) session_start();
8348: 					if (!empty($_SESSION[UID]))
8349: 					{
8350: 						if (empty($_COOKIE['vfoc']) 
8351: 							||$_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID])))) 
8352: 							|| empty($_SESSION['auth_time']) 
8353: 							|| time() > $_SESSION['auth_time'] + $this->constant['token_interval_for_auth_hash'])
8354: 						{
8355: 
8356: 							session_unset();
8357: 
8358: 						}
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
```

### `archive_17012026_1259/taxi/models/api.php:8367`
```php
8355: 
8356: 							session_unset();
8357: 
8358: 						}
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
```

### `archive_17012026_1259/taxi/models/api.php:8369`
```php
8357: 
8358: 						}
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
```

### `archive_17012026_1259/taxi/models/api.php:8370`
```php
8358: 						}
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
```

### `archive_17012026_1259/taxi/models/api.php:8372`
```php
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
8393: 									'u_middle' => $_SESSION['middle'],
8394: 									'u_email' => $_SESSION['email'],
```

### `archive_17012026_1259/taxi/models/api.php:8376`
```php
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
8393: 									'u_middle' => $_SESSION['middle'],
8394: 									'u_email' => $_SESSION['email'],
8395: 									'u_phone' => $_SESSION['phone'],
8396: 									'u_role' => $_SESSION['id_role'],
8397: 									'u_a_role' => $this->id_role,
8398: 									'u_check_state' => $_SESSION['id_verification_status'],
```

### `archive_17012026_1259/taxi/models/api.php:8388`
```php
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
8393: 									'u_middle' => $_SESSION['middle'],
8394: 									'u_email' => $_SESSION['email'],
8395: 									'u_phone' => $_SESSION['phone'],
8396: 									'u_role' => $_SESSION['id_role'],
8397: 									'u_a_role' => $this->id_role,
8398: 									'u_check_state' => $_SESSION['id_verification_status'],
8399: 									'u_ban' => $_SESSION['user_ban_status'],
8400: 									'u_active' => $_SESSION['active'],
8401: 									'u_photo' => $_SESSION['photo_link'],
8402: 									'u_birthday' => $_SESSION['birthday_date'],
8403: 									'u_lang' => $_SESSION['id_lang'],
8404: 									'u_currency' => $_SESSION['currency'],
8405: 									'u_gps_software' => $_SESSION['id_navigation']
8406: 								)
8407: 			);
8408: 
8409: 		}
8410: 
```

### `archive_17012026_1259/taxi/models/api.php:13693`
```php
13681: 
13682: 			$q = query("COMMIT");
13683: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
13684: 	
13685: 			return array(
13686: 				'code' 		=>	'200',
13687: 				'status' 	=>	'success'
13688: 			);
13689: 		}
13690: 
13691: 		public function addPush($id_user = "", $id_push = "")
13692: 		{
13693: 			if (empty($_POST['token']) || empty($_POST['u_hash'])) return $this->showError('404', 'error', 'unauthorized access');
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
13710: 		}
13711: 
13712: 		public function createDropboxFile($file = '')
13713: 		{
13714: 			if (empty($_SESSION[UID])) {
13715: 				return $this->showError('404', 'error', 'unauthorized access');
```

### `archive_17012026_1259/taxi/models/api.php:13694`
```php
13682: 			$q = query("COMMIT");
13683: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
13684: 	
13685: 			return array(
13686: 				'code' 		=>	'200',
13687: 				'status' 	=>	'success'
13688: 			);
13689: 		}
13690: 
13691: 		public function addPush($id_user = "", $id_push = "")
13692: 		{
13693: 			if (empty($_POST['token']) || empty($_POST['u_hash'])) return $this->showError('404', 'error', 'unauthorized access');
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
13710: 		}
13711: 
13712: 		public function createDropboxFile($file = '')
13713: 		{
13714: 			if (empty($_SESSION[UID])) {
13715: 				return $this->showError('404', 'error', 'unauthorized access');
13716: 			}
```

### `archive_17012026_1259/taxi/models/api.php:13702`
```php
13690: 
13691: 		public function addPush($id_user = "", $id_push = "")
13692: 		{
13693: 			if (empty($_POST['token']) || empty($_POST['u_hash'])) return $this->showError('404', 'error', 'unauthorized access');
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
13710: 		}
13711: 
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
```

### `archive_17012026_1259/taxi/models/api.php:20744`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20748`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20753`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20763`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20789`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20793`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20797`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20801`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20808`
```php
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
20827: 
20828: 		public function selectContact($id_contact_item = NULL, $id_user = NULL, $id_owner_type = NULL, $id_contact_type = NULL, $owner_types = array(), $langs = array())
20829: 		{	
20830: 			if (empty($_SESSION[UID])) {
```

### `archive_17012026_1259/taxi/models/api.php:20812`
```php
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
20827: 
20828: 		public function selectContact($id_contact_item = NULL, $id_user = NULL, $id_owner_type = NULL, $id_contact_type = NULL, $owner_types = array(), $langs = array())
20829: 		{	
20830: 			if (empty($_SESSION[UID])) {
20831: 				return $this->showError('404', 'error', 'unauthorized access');
20832: 			}
20833: 
20834: 			$sql_list = array();
```

### `archive_17012026_1259/taxi/models/api.php:20817`
```php
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
20827: 
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
```

### `archive_17012026_1259/taxi/models/api.php:20857`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:20869`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21163`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21164`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21367`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:21368`
```php
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
```

### `archive_17012026_1259/taxi/models/m_functions.php:3407`
```php
3395: 		$hmac = substr($c, $ivlen, $sha2len=32);
3396: 		$ciphertext_raw = substr($c, $ivlen+$sha2len);
3397: 		$original_plaintext = openssl_decrypt($ciphertext_raw, $cipher, $key, $options=OPENSSL_RAW_DATA, $iv);
3398: 		$calcmac = hash_hmac('sha256', $ciphertext_raw, $key, $as_binary=true);
3399: 		if (@hash_equals($hmac, $calcmac)) return $original_plaintext; else return false;
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
3429: 			} 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3409`
```php
3397: 		$original_plaintext = openssl_decrypt($ciphertext_raw, $cipher, $key, $options=OPENSSL_RAW_DATA, $iv);
3398: 		$calcmac = hash_hmac('sha256', $ciphertext_raw, $key, $as_binary=true);
3399: 		if (@hash_equals($hmac, $calcmac)) return $original_plaintext; else return false;
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
3429: 			} 
3430: 			else 
3431: 			{
```

### `archive_17012026_1259/taxi/models/m_functions.php:3412`
```php
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
3429: 			} 
3430: 			else 
3431: 			{
3432: 				$res = $str1 ^ $str2;
3433: 				$ret = 0;
3434: 				for($i = strlen($res) - 1; $i >= 0; $i--) $ret |= ord($res[$i]);
```

### `archive_17012026_1259/taxi/models/m_functions.php:3417`
```php
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
3429: 			} 
3430: 			else 
3431: 			{
3432: 				$res = $str1 ^ $str2;
3433: 				$ret = 0;
3434: 				for($i = strlen($res) - 1; $i >= 0; $i--) $ret |= ord($res[$i]);
3435: 				return !$ret;
3436: 			}
3437: 		}
3438: 	}
3439: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3446`
```php
3434: 				for($i = strlen($res) - 1; $i >= 0; $i--) $ret |= ord($res[$i]);
3435: 				return !$ret;
3436: 			}
3437: 		}
3438: 	}
3439: 
3440: 	function get_token($id_user = '')
3441: 	{
3442: 		$q = query("BEGIN");
3443: 		if ($q === false) return 0;
3444: 		
3445: 		$s = "SELECT 
3446: 				`token`
3447: 			FROM `token`
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
```

### `archive_17012026_1259/taxi/models/m_functions.php:3447`
```php
3435: 				return !$ret;
3436: 			}
3437: 		}
3438: 	}
3439: 
3440: 	function get_token($id_user = '')
3441: 	{
3442: 		$q = query("BEGIN");
3443: 		if ($q === false) return 0;
3444: 		
3445: 		$s = "SELECT 
3446: 				`token`
3447: 			FROM `token`
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
```

### `archive_17012026_1259/taxi/models/m_functions.php:3460`
```php
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
```

### `archive_17012026_1259/taxi/models/m_functions.php:3462`
```php
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3468`
```php
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
```

### `archive_17012026_1259/taxi/models/m_functions.php:3469`
```php
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
```

### `archive_17012026_1259/taxi/models/m_functions.php:3471`
```php
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
```

### `archive_17012026_1259/taxi/models/m_functions.php:3482`
```php
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
```

### `archive_17012026_1259/taxi/models/m_functions.php:3486`
```php
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
```

### `archive_17012026_1259/taxi/models/m_functions.php:3489`
```php
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
3509: 	{
3510: 		$date_str = empty($date_str) || $date_str === '0000-00-00 00:00:00' ? NULL : $date_str . MYSQL_TIME_ZONE;
3511: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3490`
```php
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
3509: 	{
3510: 		$date_str = empty($date_str) || $date_str === '0000-00-00 00:00:00' ? NULL : $date_str . MYSQL_TIME_ZONE;
3511: 	}
3512: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3492`
```php
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
3509: 	{
3510: 		$date_str = empty($date_str) || $date_str === '0000-00-00 00:00:00' ? NULL : $date_str . MYSQL_TIME_ZONE;
3511: 	}
3512: 
3513: 	function replaceIfBlocks($key = "", $value = "", $str = "")
3514: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:3498`
```php
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
3509: 	{
3510: 		$date_str = empty($date_str) || $date_str === '0000-00-00 00:00:00' ? NULL : $date_str . MYSQL_TIME_ZONE;
3511: 	}
3512: 
3513: 	function replaceIfBlocks($key = "", $value = "", $str = "")
3514: 	{
3515: 		if (empty($key))
3516: 		{
3517: 			return $str;
3518: 		}
3519: 		else
3520: 		{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4732`
```php
4720: 
4721: 	function send_msg_to_whatsapp($phone = "", $body = "", $url = "", $key = "")
4722: 	{
4723: 		if (!empty(taxi::$data_private['site_constants']['whatsapp_send_message_url']['value'])) $url = trim(taxi::$data_private['site_constants']['whatsapp_send_message_url']['value']);
4724: 		if (!empty(taxi::$data_private['site_constants']['whatsapp_send_message_key']['value'])) $key = trim(taxi::$data_private['site_constants']['whatsapp_send_message_key']['value']);
4725: 		if (empty($phone) || empty($body) || empty($url) || empty($key)) return false;
4726: 		$postvars = "phone=" . urlencode($phone) . "&code=" . urlencode($body);
4727: 		$c = curl_init();
4728: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4729: 		curl_setopt($c,CURLOPT_URL, $url);
4730: 		curl_setopt($c,CURLOPT_POST, 1);
4731: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4732: 		$headers = array(
4733: 			"x-api-key: $key"
4734: 		);
4735: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4736: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4737: 		$contents = curl_exec($c);
4738: 		curl_close($c);
4739: 		if ($contents) return $contents; else return false;
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
4754: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4735`
```php
4723: 		if (!empty(taxi::$data_private['site_constants']['whatsapp_send_message_url']['value'])) $url = trim(taxi::$data_private['site_constants']['whatsapp_send_message_url']['value']);
4724: 		if (!empty(taxi::$data_private['site_constants']['whatsapp_send_message_key']['value'])) $key = trim(taxi::$data_private['site_constants']['whatsapp_send_message_key']['value']);
4725: 		if (empty($phone) || empty($body) || empty($url) || empty($key)) return false;
4726: 		$postvars = "phone=" . urlencode($phone) . "&code=" . urlencode($body);
4727: 		$c = curl_init();
4728: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4729: 		curl_setopt($c,CURLOPT_URL, $url);
4730: 		curl_setopt($c,CURLOPT_POST, 1);
4731: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4732: 		$headers = array(
4733: 			"x-api-key: $key"
4734: 		);
4735: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4736: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4737: 		$contents = curl_exec($c);
4738: 		curl_close($c);
4739: 		if ($contents) return $contents; else return false;
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
4754: 	{
4755: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4756: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4757: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4786`
```php
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
4791: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4792: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4793: 			$response = curl_exec($c);
4794: 			curl_close($c);
4795: 			if (empty($response)) return array('error'=>'upload failed');
4796: 			@$response = json_decode($response,true);
4797: 			if (empty($response) || !is_array($response)) 
4798: 			{
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
```

### `archive_17012026_1259/taxi/models/m_functions.php:4787`
```php
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
4791: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4792: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4793: 			$response = curl_exec($c);
4794: 			curl_close($c);
4795: 			if (empty($response)) return array('error'=>'upload failed');
4796: 			@$response = json_decode($response,true);
4797: 			if (empty($response) || !is_array($response)) 
4798: 			{
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
```

### `archive_17012026_1259/taxi/models/m_functions.php:4791`
```php
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
4791: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4792: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4793: 			$response = curl_exec($c);
4794: 			curl_close($c);
4795: 			if (empty($response)) return array('error'=>'upload failed');
4796: 			@$response = json_decode($response,true);
4797: 			if (empty($response) || !is_array($response)) 
4798: 			{
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
```

### `archive_17012026_1259/taxi/models/m_functions.php:4848`
```php
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4849`
```php
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
4871: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
```

### `archive_17012026_1259/taxi/models/m_functions.php:4852`
```php
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
4871: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4872: 			$response = curl_exec($c);
4873: 			curl_close($c);
4874: 			if ($response === true)
```

### `archive_17012026_1259/taxi/models/m_functions.php:4853`
```php
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
4871: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4872: 			$response = curl_exec($c);
4873: 			curl_close($c);
4874: 			if ($response === true)
4875: 			{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4865`
```php
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
4871: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4872: 			$response = curl_exec($c);
4873: 			curl_close($c);
4874: 			if ($response === true)
4875: 			{
4876: 				exit;
4877: 			}
4878: 			if (empty($response)) return array('error'=>'download failed');
4879: 			@$response = json_decode($response,true);
4880: 			if (empty($response) || !is_array($response)) 
4881: 			{
4882: 				 return array('error'=>'download response error');
4883: 			}
4884: 			if (!empty($response['error']))
4885: 			{
4886: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4887: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4869`
```php
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
4866: 				return $len;	
4867: 			});
4868: 			header("Content-Type: $type");
4869: 			header("Access-Control-Expose-Headers: Content-disposition");
4870: 			set_header_content_disposition($filename_upload);
4871: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4872: 			$response = curl_exec($c);
4873: 			curl_close($c);
4874: 			if ($response === true)
4875: 			{
4876: 				exit;
4877: 			}
4878: 			if (empty($response)) return array('error'=>'download failed');
4879: 			@$response = json_decode($response,true);
4880: 			if (empty($response) || !is_array($response)) 
4881: 			{
4882: 				 return array('error'=>'download response error');
4883: 			}
4884: 			if (!empty($response['error']))
4885: 			{
4886: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4887: 				{
4888: 					if ($update == true)
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
```

### `archive_17012026_1259/taxi/models/m_functions.php:4918`
```php
4906: 		}		
4907: 	}
4908: 
4909: 	function update_dropbox($code = '')
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4912: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
4914: 		
4915: 		$postvars = "code=" . urlencode($code) . "&grant_type=authorization_code&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4916: 		$c = curl_init();
4917: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4918: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4919: 		curl_setopt($c,CURLOPT_POST, 1);
4920: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4921: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4922: 		$headers = array(
4923: 			"Content-Type: application/x-www-form-urlencoded"
4924: 		);
4925: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4926: 		$response = curl_exec($c);
4927: 		curl_close($c);
4928: 		if (empty($response)) return array('error'=>'update failed');
4929: 		@$response = json_decode($response,true);
4930: 		if (empty($response) || !is_array($response)) 
4931: 		{
4932: 			 return array('error'=>'update response error');
4933: 		}
4934: 		if (!empty($response['error']))
4935: 		{
4936: 			if (!empty($response['error_description']) && substr($response['error_description'],0,16) == 'code has expired')
4937: 			{
4938: 				return array('error'=>'code expired');
4939: 			}
4940: 			else
```

### `archive_17012026_1259/taxi/models/m_functions.php:4922`
```php
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4912: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
4914: 		
4915: 		$postvars = "code=" . urlencode($code) . "&grant_type=authorization_code&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4916: 		$c = curl_init();
4917: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4918: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4919: 		curl_setopt($c,CURLOPT_POST, 1);
4920: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4921: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4922: 		$headers = array(
4923: 			"Content-Type: application/x-www-form-urlencoded"
4924: 		);
4925: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4926: 		$response = curl_exec($c);
4927: 		curl_close($c);
4928: 		if (empty($response)) return array('error'=>'update failed');
4929: 		@$response = json_decode($response,true);
4930: 		if (empty($response) || !is_array($response)) 
4931: 		{
4932: 			 return array('error'=>'update response error');
4933: 		}
4934: 		if (!empty($response['error']))
4935: 		{
4936: 			if (!empty($response['error_description']) && substr($response['error_description'],0,16) == 'code has expired')
4937: 			{
4938: 				return array('error'=>'code expired');
4939: 			}
4940: 			else
4941: 			{
4942: 				return array('error'=>'update error');
4943: 			}
4944: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:4925`
```php
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
4914: 		
4915: 		$postvars = "code=" . urlencode($code) . "&grant_type=authorization_code&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4916: 		$c = curl_init();
4917: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4918: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4919: 		curl_setopt($c,CURLOPT_POST, 1);
4920: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4921: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4922: 		$headers = array(
4923: 			"Content-Type: application/x-www-form-urlencoded"
4924: 		);
4925: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4926: 		$response = curl_exec($c);
4927: 		curl_close($c);
4928: 		if (empty($response)) return array('error'=>'update failed');
4929: 		@$response = json_decode($response,true);
4930: 		if (empty($response) || !is_array($response)) 
4931: 		{
4932: 			 return array('error'=>'update response error');
4933: 		}
4934: 		if (!empty($response['error']))
4935: 		{
4936: 			if (!empty($response['error_description']) && substr($response['error_description'],0,16) == 'code has expired')
4937: 			{
4938: 				return array('error'=>'code expired');
4939: 			}
4940: 			else
4941: 			{
4942: 				return array('error'=>'update error');
4943: 			}
4944: 		}
4945: 		if (empty($response['refresh_token'])) return array('error'=>'empty refresh_token');
4946: 		$refresh_token = $response['refresh_token'];
4947: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:4988`
```php
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
```

### `archive_17012026_1259/taxi/models/m_functions.php:4991`
```php
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4994`
```php
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
5014: 	}
5015: 
5016: 	function fields_for_langs($langs = array(), &$arr = array(), $out_prefix = '', $db_prefix = '', $null_on = false)
```

### `archive_17012026_1259/taxi/models/m_functions.php:5100`
```php
5088: 							'site'			=>		url('',0,1),
5089: 							'key'			=>		$key,
5090: 							'title'			=>		$title,
5091: 							'duration'		=>		$duration
5092: 		);
5093: 		if (!empty($email)) $arr['email'] = $email;
5094: 		$c = curl_init();
5095: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5096: 		curl_setopt($c,CURLOPT_URL, $url);
5097: 		curl_setopt($c,CURLOPT_POST, 1);
5098: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5099: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5100: 		$headers = array(
5101: 			"Content-Type: application/json"
5102: 		);
5103: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5104: 		$response = curl_exec($c);
5105: 		curl_close($c);
5106: 		if (empty($response)) return array('error'=>'empty response');
5107: 		$response_str = $response;
5108: 		@$response = json_decode($response,true);
5109: 		if (empty($response) || !is_array($response)) 
5110: 		{
5111: 			 return array('error'=>'response error:' . $response_str);
5112: 		}
5113: 		if (!empty($response['error']))
5114: 		{
5115: 			return array('error'=>array('response with error',$response));
5116: 		}
5117: 		if (empty($response['url'])) return array('error'=>array('empty url',$response));
5118: 		return array($response['url'],$duration);
5119: 	}
5120: 
5121: 	function get_stripe_status($order = '')
5122: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:5103`
```php
5091: 							'duration'		=>		$duration
5092: 		);
5093: 		if (!empty($email)) $arr['email'] = $email;
5094: 		$c = curl_init();
5095: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5096: 		curl_setopt($c,CURLOPT_URL, $url);
5097: 		curl_setopt($c,CURLOPT_POST, 1);
5098: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5099: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5100: 		$headers = array(
5101: 			"Content-Type: application/json"
5102: 		);
5103: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5104: 		$response = curl_exec($c);
5105: 		curl_close($c);
5106: 		if (empty($response)) return array('error'=>'empty response');
5107: 		$response_str = $response;
5108: 		@$response = json_decode($response,true);
5109: 		if (empty($response) || !is_array($response)) 
5110: 		{
5111: 			 return array('error'=>'response error:' . $response_str);
5112: 		}
5113: 		if (!empty($response['error']))
5114: 		{
5115: 			return array('error'=>array('response with error',$response));
5116: 		}
5117: 		if (empty($response['url'])) return array('error'=>array('empty url',$response));
5118: 		return array($response['url'],$duration);
5119: 	}
5120: 
5121: 	function get_stripe_status($order = '')
5122: 	{
5123: 		if (empty($order)) return array('error'=>'empty b_id');
5124: 		if (!empty(taxi::$data_private['site_constants']['stripe_status_url']['value'])) $url = trim(taxi::$data_private['site_constants']['stripe_status_url']['value']);
5125: 		if (!empty(taxi::$data_private['site_constants']['stripe_secret_key']['value'])) $key = trim(taxi::$data_private['site_constants']['stripe_secret_key']['value']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:5142`
```php
5130: 							'site'			=>		url('',0,1),
5131: 		);
5132: 		$c = $arr['c'];
5133: 		$site = $arr['site'];
5134: 		$hash = md5("$order$c$site$key");
5135: 		$arr['hash'] = $hash;
5136: 		$c = curl_init();
5137: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5138: 		curl_setopt($c,CURLOPT_URL, $url);
5139: 		curl_setopt($c,CURLOPT_POST, 1);
5140: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5141: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5142: 		$headers = array(
5143: 			"Content-Type: application/json"
5144: 		);
5145: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5146: 		$response = curl_exec($c);
5147: 		curl_close($c);
5148: 		if (empty($response)) return array('error'=>'empty response');
5149: 		$response_str = $response;
5150: 		@$response = json_decode($response,true);
5151: 		if (empty($response) || !is_array($response)) 
5152: 		{
5153: 			 return array('error'=>'response error:' . $response_str);
5154: 		}
5155: 		if (!empty($response['error']))
5156: 		{
5157: 			return array('error'=>array('response with error',$response));
5158: 		}
5159: 		return $response;
5160: 	}
5161: 	
5162: 	function cancel_stripe_link($order = '')
5163: 	{
5164: 		if (empty($order)) return array('error'=>'empty b_id');
```

### `archive_17012026_1259/taxi/models/m_functions.php:5145`
```php
5133: 		$site = $arr['site'];
5134: 		$hash = md5("$order$c$site$key");
5135: 		$arr['hash'] = $hash;
5136: 		$c = curl_init();
5137: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5138: 		curl_setopt($c,CURLOPT_URL, $url);
5139: 		curl_setopt($c,CURLOPT_POST, 1);
5140: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5141: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5142: 		$headers = array(
5143: 			"Content-Type: application/json"
5144: 		);
5145: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5146: 		$response = curl_exec($c);
5147: 		curl_close($c);
5148: 		if (empty($response)) return array('error'=>'empty response');
5149: 		$response_str = $response;
5150: 		@$response = json_decode($response,true);
5151: 		if (empty($response) || !is_array($response)) 
5152: 		{
5153: 			 return array('error'=>'response error:' . $response_str);
5154: 		}
5155: 		if (!empty($response['error']))
5156: 		{
5157: 			return array('error'=>array('response with error',$response));
5158: 		}
5159: 		return $response;
5160: 	}
5161: 	
5162: 	function cancel_stripe_link($order = '')
5163: 	{
5164: 		if (empty($order)) return array('error'=>'empty b_id');
5165: 		if (!empty(taxi::$data_private['site_constants']['stripe_cancel_url']['value'])) $url = trim(taxi::$data_private['site_constants']['stripe_cancel_url']['value']);
5166: 		if (!empty(taxi::$data_private['site_constants']['stripe_secret_key']['value'])) $key = trim(taxi::$data_private['site_constants']['stripe_secret_key']['value']);
5167: 		if (empty($url) || empty($key)) return array('error'=>'empty parameters');
```

### `archive_17012026_1259/taxi/models/m_functions.php:5183`
```php
5171: 							'site'			=>		url('',0,1),
5172: 		);
5173: 		$c = $arr['c'];
5174: 		$site = $arr['site'];
5175: 		$hash = md5("$order$c$site$key");
5176: 		$arr['hash'] = $hash;
5177: 		$c = curl_init();
5178: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5179: 		curl_setopt($c,CURLOPT_URL, $url);
5180: 		curl_setopt($c,CURLOPT_POST, 1);
5181: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5182: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5183: 		$headers = array(
5184: 			"Content-Type: application/json"
5185: 		);
5186: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5187: 		$response = curl_exec($c);
5188: 		curl_close($c);
5189: 		if (empty($response)) return array('error'=>'empty response');
5190: 		$response_str = $response;
5191: 		@$response = json_decode($response,true);
5192: 		if (empty($response) || !is_array($response)) 
5193: 		{
5194: 			 return array('error'=>'response error:' . $response_str);
5195: 		}
5196: 		if (!empty($response['error']))
5197: 		{
5198: 			return array('error'=>array('response with error',$response['error']));
5199: 		}
5200: 		if (empty($response['status'])) return array('error'=>array('empty status',$response));
5201: 		return $response;
5202: 	}
5203: 
5204: 	function pre_debug($str = "")
5205: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:5186`
```php
5174: 		$site = $arr['site'];
5175: 		$hash = md5("$order$c$site$key");
5176: 		$arr['hash'] = $hash;
5177: 		$c = curl_init();
5178: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
5179: 		curl_setopt($c,CURLOPT_URL, $url);
5180: 		curl_setopt($c,CURLOPT_POST, 1);
5181: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
5182: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
5183: 		$headers = array(
5184: 			"Content-Type: application/json"
5185: 		);
5186: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
5187: 		$response = curl_exec($c);
5188: 		curl_close($c);
5189: 		if (empty($response)) return array('error'=>'empty response');
5190: 		$response_str = $response;
5191: 		@$response = json_decode($response,true);
5192: 		if (empty($response) || !is_array($response)) 
5193: 		{
5194: 			 return array('error'=>'response error:' . $response_str);
5195: 		}
5196: 		if (!empty($response['error']))
5197: 		{
5198: 			return array('error'=>array('response with error',$response['error']));
5199: 		}
5200: 		if (empty($response['status'])) return array('error'=>array('empty status',$response));
5201: 		return $response;
5202: 	}
5203: 
5204: 	function pre_debug($str = "")
5205: 	{
5206: 		echo '<pre>' . htmlentities(var_export($str,true)). '</pre>';
5207: 		return true;
5208: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:6313`
```php
6301:             pack('V', $oldOffset) . //offset to start of central dir
6302:             "\x00\x00"; //.zip file comment length
6303: 
6304:         $data = implode('', $datasec);
6305: 
6306:         return $data . $header;
6307:     }
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
6330: 			$msg = [];
6331: 			if ($subject !== "") $msg[] = $subject;
6332: 			if ($body !== "") $msg[] = $body;
6333: 			$arr['msg'] = implode("\n",$msg);
6334: 		}
6335: 		else
```

### `archive_17012026_1259/taxi/models/m_functions.php:6314`
```php
6302:             "\x00\x00"; //.zip file comment length
6303: 
6304:         $data = implode('', $datasec);
6305: 
6306:         return $data . $header;
6307:     }
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
6330: 			$msg = [];
6331: 			if ($subject !== "") $msg[] = $subject;
6332: 			if ($body !== "") $msg[] = $body;
6333: 			$arr['msg'] = implode("\n",$msg);
6334: 		}
6335: 		else
6336: 		{
```

### `archive_17012026_1259/taxi/models/m_functions.php:6316`
```php
6304:         $data = implode('', $datasec);
6305: 
6306:         return $data . $header;
6307:     }
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
6330: 			$msg = [];
6331: 			if ($subject !== "") $msg[] = $subject;
6332: 			if ($body !== "") $msg[] = $body;
6333: 			$arr['msg'] = implode("\n",$msg);
6334: 		}
6335: 		else
6336: 		{
6337: 			$arr['code'] = $code;
6338: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:6780`
```php
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
6785: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
6786: 			$response = curl_exec($c);
6787: 			curl_close($c);
6788: 			if (empty($response)) return array('error'=>'upload failed');
6789: 			@$response = json_decode($response,true);
6790: 			if (empty($response) || !is_array($response)) 
6791: 			{
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
```

### `archive_17012026_1259/taxi/models/m_functions.php:6781`
```php
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
6785: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
6786: 			$response = curl_exec($c);
6787: 			curl_close($c);
6788: 			if (empty($response)) return array('error'=>'upload failed');
6789: 			@$response = json_decode($response,true);
6790: 			if (empty($response) || !is_array($response)) 
6791: 			{
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
```

### `archive_17012026_1259/taxi/models/m_functions.php:6784`
```php
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
6785: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
6786: 			$response = curl_exec($c);
6787: 			curl_close($c);
6788: 			if (empty($response)) return array('error'=>'upload failed');
6789: 			@$response = json_decode($response,true);
6790: 			if (empty($response) || !is_array($response)) 
6791: 			{
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
```

### `archive_17012026_1259/taxi/models/m_functions.php:7049`
```php
7037: 		if (!empty(taxi::$data_private['site_constants']['telegram_send_message_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_send_message_url']['value']);
7038: 		if (!empty(taxi::$data_private['site_constants']['telegram_send_message_key']['value'])) $key = trim(taxi::$data_private['site_constants']['telegram_send_message_key']['value']);
7039: 		if (empty($id) || empty($body) || empty($url) || empty($key)) return false;
7040: 		$post_data = array(
7041: 			'user_id'	=>	$id,
7042: 			'code'		=>	$body
7043: 		);
7044: 		$c = curl_init();
7045: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7046: 		curl_setopt($c,CURLOPT_URL, $url);
7047: 		curl_setopt($c,CURLOPT_POST, 1);
7048: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($post_data));
7049: 		$headers = array(
7050: 			"Content-Type: application/json",
7051: 			"x-api-key: $key"
7052: 		);
7053: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7054: 		$contents = curl_exec($c);
7055: 		curl_close($c);
7056: 		if (!$contents) return false;
7057: 		$contents = json_decode($contents,true);
7058: 		if (!$contents) return false;
7059: 		if ($contents['status'] !== 'success') return false;
7060: 		return $contents;
7061: 	}
7062: 
7063: 	function send_sms($phone = "", $body = "", $constants = array())
7064: 	{
7065: 		if ($body === "") return array('error'=>'empty input data');
7066: 		if ($phone === "") return array('error'=>'empty phone');
7067: 		if (!empty(taxi::$data_private['site_constants']['sms_ru_send_url']['value'])) $url = trim(taxi::$data_private['site_constants']['sms_ru_send_url']['value']);
7068: 		if (!empty(taxi::$data_private['site_constants']['sms_ru_api_id']['value'])) $api_id = trim(taxi::$data_private['site_constants']['sms_ru_api_id']['value']);
7069: 		if (empty($url) || empty($api_id)) return array('error'=>'empty parameters');
7070: 		$arr = array(
7071: 			'api_id'	=>	$api_id,
```

### `archive_17012026_1259/taxi/models/m_functions.php:7053`
```php
7041: 			'user_id'	=>	$id,
7042: 			'code'		=>	$body
7043: 		);
7044: 		$c = curl_init();
7045: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7046: 		curl_setopt($c,CURLOPT_URL, $url);
7047: 		curl_setopt($c,CURLOPT_POST, 1);
7048: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($post_data));
7049: 		$headers = array(
7050: 			"Content-Type: application/json",
7051: 			"x-api-key: $key"
7052: 		);
7053: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7054: 		$contents = curl_exec($c);
7055: 		curl_close($c);
7056: 		if (!$contents) return false;
7057: 		$contents = json_decode($contents,true);
7058: 		if (!$contents) return false;
7059: 		if ($contents['status'] !== 'success') return false;
7060: 		return $contents;
7061: 	}
7062: 
7063: 	function send_sms($phone = "", $body = "", $constants = array())
7064: 	{
7065: 		if ($body === "") return array('error'=>'empty input data');
7066: 		if ($phone === "") return array('error'=>'empty phone');
7067: 		if (!empty(taxi::$data_private['site_constants']['sms_ru_send_url']['value'])) $url = trim(taxi::$data_private['site_constants']['sms_ru_send_url']['value']);
7068: 		if (!empty(taxi::$data_private['site_constants']['sms_ru_api_id']['value'])) $api_id = trim(taxi::$data_private['site_constants']['sms_ru_api_id']['value']);
7069: 		if (empty($url) || empty($api_id)) return array('error'=>'empty parameters');
7070: 		$arr = array(
7071: 			'api_id'	=>	$api_id,
7072: 			'to'		=>	$phone,
7073: 			'msg'		=>	$body,
7074: 			'json'		=>	1,
7075: //			'test'		=>	1
```

### `archive_17012026_1259/taxi/models/m_functions.php:7166`
```php
7154: 		}
7155: 		if (!empty($email)) $arr['metadata']['email'] = $email;
7156: 		if (!empty($id_payment)) $arr['metadata']['id_payment'] = $id_payment;
7157: 		$c = curl_init();
7158: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7159: 		curl_setopt($c,CURLOPT_URL, $url);
7160: 		curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
7161: 		curl_setopt($c,CURLOPT_USERPWD, "$shopid:$key");
7162: 		curl_setopt($c,CURLOPT_POST, 1);
7163: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
7164: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
7165: 		$idempotence = empty($id_payment) ? time() : $id_payment;
7166: 		$headers = array(
7167: 			"Content-Type: application/json",
7168: 			"Idempotence-Key: $idempotence"
7169: 		);
7170: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7171: 		$response = curl_exec($c);
7172: 		curl_close($c);
7173: 		if (empty($response)) return array('error'=>'empty response');
7174: 		$response_str = $response;
7175: 		@$response = json_decode($response,true);
7176: 		if (empty($response) || !is_array($response)) 
7177: 		{
7178: 			 return array('error'=>'response error:' . $response_str);
7179: 		}
7180: 		if (!empty($response['type']) && $response['type'] == 'error')
7181: 		{
7182: 			return array('error'=>array('response with error',$response));
7183: 		}
7184: 		return array('id'=>$response['id'],'status'=>$response['status'],'paid'=>$response['paid'],'confirmation_url'=>$response['confirmation']['confirmation_url']);
7185: 	}
7186: 
7187: 	function yookassa_get_payment($id_outer = '')
7188: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:7170`
```php
7158: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7159: 		curl_setopt($c,CURLOPT_URL, $url);
7160: 		curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
7161: 		curl_setopt($c,CURLOPT_USERPWD, "$shopid:$key");
7162: 		curl_setopt($c,CURLOPT_POST, 1);
7163: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
7164: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
7165: 		$idempotence = empty($id_payment) ? time() : $id_payment;
7166: 		$headers = array(
7167: 			"Content-Type: application/json",
7168: 			"Idempotence-Key: $idempotence"
7169: 		);
7170: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7171: 		$response = curl_exec($c);
7172: 		curl_close($c);
7173: 		if (empty($response)) return array('error'=>'empty response');
7174: 		$response_str = $response;
7175: 		@$response = json_decode($response,true);
7176: 		if (empty($response) || !is_array($response)) 
7177: 		{
7178: 			 return array('error'=>'response error:' . $response_str);
7179: 		}
7180: 		if (!empty($response['type']) && $response['type'] == 'error')
7181: 		{
7182: 			return array('error'=>array('response with error',$response));
7183: 		}
7184: 		return array('id'=>$response['id'],'status'=>$response['status'],'paid'=>$response['paid'],'confirmation_url'=>$response['confirmation']['confirmation_url']);
7185: 	}
7186: 
7187: 	function yookassa_get_payment($id_outer = '')
7188: 	{
7189: 		if (empty($id_outer)) return array('error'=>'id');	
7190: 		if (!empty(taxi::$data_private['site_constants']['yookassa_payment_url']['value'])) $url = trim(taxi::$data_private['site_constants']['yookassa_payment_url']['value']);
7191: 		if (!empty(taxi::$data_private['site_constants']['yookassa_secret_key']['value'])) $key = trim(taxi::$data_private['site_constants']['yookassa_secret_key']['value']);
7192: 		if (!empty(taxi::$data_private['site_constants']['yookassa_shopid_key']['value'])) $shopid = trim(taxi::$data_private['site_constants']['yookassa_shopid_key']['value']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:7262`
```php
7250: 				$arr['metadata'][$arr_key] = $arr_val;
7251: 			}
7252: 		}
7253: 		$c = curl_init();
7254: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7255: 		curl_setopt($c,CURLOPT_URL, $url);
7256: 		curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
7257: 		curl_setopt($c,CURLOPT_USERPWD, "$agentid:$key");
7258: 		curl_setopt($c,CURLOPT_POST, 1);
7259: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
7260: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
7261: 		$idempotence = time();
7262: 		$headers = array(
7263: 			"Content-Type: application/json",
7264: 			"Idempotence-Key: $idempotence"
7265: 		);
7266: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7267: 		$response = curl_exec($c);
7268: 		curl_close($c);
7269: 		if (empty($response)) return array('error'=>'empty response');
7270: 		$response_str = $response;
7271: 		@$response = json_decode($response,true);
7272: 		if (empty($response) || !is_array($response)) 
7273: 		{
7274: 			 return array('error'=>'response error:' . $response_str);
7275: 		}
7276: 		if (!empty($response['type']) && $response['type'] == 'error')
7277: 		{
7278: 			return array('error'=>array('response with error',$response));
7279: 		}
7280: 		return array('id'=>$response['id'],'status'=>$response['status'],'payout_destination'=>$response['payout_destination']);
7281: 	}
7282: 
7283: 	function yookassa_get_payout($id_outer = '')
7284: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:7266`
```php
7254: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7255: 		curl_setopt($c,CURLOPT_URL, $url);
7256: 		curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
7257: 		curl_setopt($c,CURLOPT_USERPWD, "$agentid:$key");
7258: 		curl_setopt($c,CURLOPT_POST, 1);
7259: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
7260: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
7261: 		$idempotence = time();
7262: 		$headers = array(
7263: 			"Content-Type: application/json",
7264: 			"Idempotence-Key: $idempotence"
7265: 		);
7266: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
7267: 		$response = curl_exec($c);
7268: 		curl_close($c);
7269: 		if (empty($response)) return array('error'=>'empty response');
7270: 		$response_str = $response;
7271: 		@$response = json_decode($response,true);
7272: 		if (empty($response) || !is_array($response)) 
7273: 		{
7274: 			 return array('error'=>'response error:' . $response_str);
7275: 		}
7276: 		if (!empty($response['type']) && $response['type'] == 'error')
7277: 		{
7278: 			return array('error'=>array('response with error',$response));
7279: 		}
7280: 		return array('id'=>$response['id'],'status'=>$response['status'],'payout_destination'=>$response['payout_destination']);
7281: 	}
7282: 
7283: 	function yookassa_get_payout($id_outer = '')
7284: 	{
7285: 		if (empty($id_outer)) return array('error'=>'id');	
7286: 		if (!empty(taxi::$data_private['site_constants']['yookassa_payout_url']['value'])) $url = trim(taxi::$data_private['site_constants']['yookassa_payout_url']['value']);
7287: 		if (!empty(taxi::$data_private['site_constants']['yookassa_agentid_secret_key']['value'])) $key = trim(taxi::$data_private['site_constants']['yookassa_agentid_secret_key']['value']);
7288: 		if (!empty(taxi::$data_private['site_constants']['yookassa_agentid_key']['value'])) $agentid = trim(taxi::$data_private['site_constants']['yookassa_agentid_key']['value']);
```

### `archive_17012026_1259/taxi/models/token.php:2`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
23: 	{
24: 		if ($_GET['route'] !== 'api')
```

### `archive_17012026_1259/taxi/models/token.php:6`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
23: 	{
24: 		if ($_GET['route'] !== 'api')
25: 		{
26: 
27: 			$_GET['route'] = 'api';
28: 			unset($_GET['par']);
```

### `archive_17012026_1259/taxi/models/token.php:9`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
23: 	{
24: 		if ($_GET['route'] !== 'api')
25: 		{
26: 
27: 			$_GET['route'] = 'api';
28: 			unset($_GET['par']);
29: 		}
30: 	} 
31: 	else 
```

### `archive_17012026_1259/taxi/stub/stripe.php:231`
```php
219: 			$postvars = array();
220: 			foreach($arr as $var_key=>$var_val)
221: 			{
222: 				$postvars[] = urlencode($var_key) . '=' . urlencode($var_val);
223: 			}
224: 			$postvars = implode('&',$postvars);
225: 			$c = curl_init();
226: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
227: 			curl_setopt($c,CURLOPT_URL, $endpoint);
228: 			curl_setopt($c,CURLOPT_POST, 1);
229: 			curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
230: 			curl_setopt($c,CURLOPT_CAINFO,'../config/cacert.pem');
231: 			$headers = array(
232: 				"Content-Type: application/x-www-form-urlencoded"
233: 			);
234: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
235: 			$response = curl_exec($c);
236: 			curl_close($c);
237: 			if (empty($response)) {echo json_encode(array('error'=>'empty response'));break;}
238: 			echo 'success';
239: 			break;
240: 	}
241: }
```

### `archive_17012026_1259/taxi/stub/stripe.php:234`
```php
222: 				$postvars[] = urlencode($var_key) . '=' . urlencode($var_val);
223: 			}
224: 			$postvars = implode('&',$postvars);
225: 			$c = curl_init();
226: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
227: 			curl_setopt($c,CURLOPT_URL, $endpoint);
228: 			curl_setopt($c,CURLOPT_POST, 1);
229: 			curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
230: 			curl_setopt($c,CURLOPT_CAINFO,'../config/cacert.pem');
231: 			$headers = array(
232: 				"Content-Type: application/x-www-form-urlencoded"
233: 			);
234: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
235: 			$response = curl_exec($c);
236: 			curl_close($c);
237: 			if (empty($response)) {echo json_encode(array('error'=>'empty response'));break;}
238: 			echo 'success';
239: 			break;
240: 	}
241: }
```

### `archive_17012026_1259/taxi/config/mime.php:93`
```php
81: 	'application/reginfo+xml'			=>	'rif',
82: 	'application/relax-ng-compact-syntax'			=>	'rnc',
83: 	'application/resource-lists+xml'			=>	'rl',
84: 	'application/resource-lists-diff+xml'			=>	'rld',
85: 	'application/rls-services+xml'			=>	'rs',
86: 	'application/rpki-ghostbusters'			=>	'gbr',
87: 	'application/rpki-manifest'			=>	'mft',
88: 	'application/rpki-roa'			=>	'roa',
89: 	'application/rsd+xml'			=>	'rsd',
90: 	'application/rss+xml'			=>	'rss',
91: 	'application/rtf'			=>	'rtf',
92: 	'application/sbml+xml'			=>	'sbml',
93: 	'application/scvp-cv-request'			=>	'scq',
94: 	'application/scvp-cv-response'			=>	'scs',
95: 	'application/scvp-vp-request'			=>	'spq',
96: 	'application/scvp-vp-response'			=>	'spp',
97: 	'application/sdp'			=>	'sdp',
98: 	'application/set-payment-initiation'			=>	'setpay',
99: 	'application/set-registration-initiation'			=>	'setreg',
100: 	'application/shf+xml'			=>	'shf',
101: 	'application/smil+xml'			=>	'smil',
102: 	'application/sparql-query'			=>	'rq',
103: 	'application/sparql-results+xml'			=>	'srx',
104: 	'application/srgs'			=>	'gram',
105: 	'application/srgs+xml'			=>	'grxml',
106: 	'application/sru+xml'			=>	'sru',
107: 	'application/ssdl+xml'			=>	'ssdl',
108: 	'application/ssml+xml'			=>	'ssml',
109: 	'application/tei+xml'			=>	'tei',
110: 	'application/thraud+xml'			=>	'tfi',
111: 	'application/timestamped-data'			=>	'tsd',
112: 	'application/vnd.3gpp.pic-bw-large'			=>	'plb',
113: 	'application/vnd.3gpp.pic-bw-small'			=>	'psb',
114: 	'application/vnd.3gpp.pic-bw-var'			=>	'pvb',
115: 	'application/vnd.3gpp2.tcap'			=>	'tcap',
```

### `archive_17012026_1259/taxi/config/mime.php:95`
```php
83: 	'application/resource-lists+xml'			=>	'rl',
84: 	'application/resource-lists-diff+xml'			=>	'rld',
85: 	'application/rls-services+xml'			=>	'rs',
86: 	'application/rpki-ghostbusters'			=>	'gbr',
87: 	'application/rpki-manifest'			=>	'mft',
88: 	'application/rpki-roa'			=>	'roa',
89: 	'application/rsd+xml'			=>	'rsd',
90: 	'application/rss+xml'			=>	'rss',
91: 	'application/rtf'			=>	'rtf',
92: 	'application/sbml+xml'			=>	'sbml',
93: 	'application/scvp-cv-request'			=>	'scq',
94: 	'application/scvp-cv-response'			=>	'scs',
95: 	'application/scvp-vp-request'			=>	'spq',
96: 	'application/scvp-vp-response'			=>	'spp',
97: 	'application/sdp'			=>	'sdp',
98: 	'application/set-payment-initiation'			=>	'setpay',
99: 	'application/set-registration-initiation'			=>	'setreg',
100: 	'application/shf+xml'			=>	'shf',
101: 	'application/smil+xml'			=>	'smil',
102: 	'application/sparql-query'			=>	'rq',
103: 	'application/sparql-results+xml'			=>	'srx',
104: 	'application/srgs'			=>	'gram',
105: 	'application/srgs+xml'			=>	'grxml',
106: 	'application/sru+xml'			=>	'sru',
107: 	'application/ssdl+xml'			=>	'ssdl',
108: 	'application/ssml+xml'			=>	'ssml',
109: 	'application/tei+xml'			=>	'tei',
110: 	'application/thraud+xml'			=>	'tfi',
111: 	'application/timestamped-data'			=>	'tsd',
112: 	'application/vnd.3gpp.pic-bw-large'			=>	'plb',
113: 	'application/vnd.3gpp.pic-bw-small'			=>	'psb',
114: 	'application/vnd.3gpp.pic-bw-var'			=>	'pvb',
115: 	'application/vnd.3gpp2.tcap'			=>	'tcap',
116: 	'application/vnd.3m.post-it-notes'			=>	'pwn',
117: 	'application/vnd.accpac.simply.aso'			=>	'aso',
```

### `archive_17012026_1259/taxi/cache/data.php:2816`
```php
2804:       2 => 'Till',
2805:     ),
2806:     'to' => 
2807:     array (
2808:       1 => 'Куда',
2809:       2 => 'To',
2810:     ),
2811:     'today' => 
2812:     array (
2813:       1 => 'Сегодня',
2814:       2 => 'Today ',
2815:     ),
2816:     'token' => 
2817:     array (
2818:       1 => 'Токен',
2819:       2 => 'Token',
2820:     ),
2821:     'tomato_tc' => 
2822:     array (
2823:       1 => 'Помидоры',
2824:       2 => 'Tomato',
2825:     ),
2826:     'tomorrow' => 
2827:     array (
2828:       1 => 'Завтра',
2829:       2 => 'Tomorrow',
2830:     ),
2831:     'took_passenger' => 
2832:     array (
2833:       1 => 'Взял пассажира',
2834:       2 => 'Took Order',
2835:     ),
2836:     'tool_box' => 
2837:     array (
2838:       1 => 'Ящик для инструмента',
```

### `archive_17012026_1259/taxi/cache/data.php:2819`
```php
2807:     array (
2808:       1 => 'Куда',
2809:       2 => 'To',
2810:     ),
2811:     'today' => 
2812:     array (
2813:       1 => 'Сегодня',
2814:       2 => 'Today ',
2815:     ),
2816:     'token' => 
2817:     array (
2818:       1 => 'Токен',
2819:       2 => 'Token',
2820:     ),
2821:     'tomato_tc' => 
2822:     array (
2823:       1 => 'Помидоры',
2824:       2 => 'Tomato',
2825:     ),
2826:     'tomorrow' => 
2827:     array (
2828:       1 => 'Завтра',
2829:       2 => 'Tomorrow',
2830:     ),
2831:     'took_passenger' => 
2832:     array (
2833:       1 => 'Взял пассажира',
2834:       2 => 'Took Order',
2835:     ),
2836:     'tool_box' => 
2837:     array (
2838:       1 => 'Ящик для инструмента',
2839:       2 => 'Tool box',
2840:     ),
2841:     'top_p' => 
```

### `archive_17012026_1259/taxi/cache/data.php:3021`
```php
3009:       2 => 'Urgently',
3010:     ),
3011:     'user_is_not_performer_error' => 
3012:     array (
3013:       1 => 'Изменение заказа доступно только после его взятия',
3014:       2 => 'User is not performer error',
3015:     ),
3016:     'use_p' => 
3017:     array (
3018:       1 => 'Использовать',
3019:       2 => 'Use',
3020:     ),
3021:     'u_hash' => 
3022:     array (
3023:       1 => 'Хэш',
3024:       2 => 'U_hash',
3025:     ),
3026:     'very_expensive' => 
3027:     array (
3028:       1 => 'Слишком дорого',
3029:       2 => 'Very expensive',
3030:     ),
3031:     'view_p' => 
3032:     array (
3033:       1 => 'Посмотреть',
3034:     ),
3035:     'vote' => 
3036:     array (
3037:       1 => 'Голосовать',
3038:       2 => 'Vote',
3039:     ),
3040:     'voter' => 
3041:     array (
3042:       1 => 'Голосующий',
3043:       2 => 'Voter',
```

### `archive_17012026_1259/taxi/cache/data.php:3024`
```php
3012:     array (
3013:       1 => 'Изменение заказа доступно только после его взятия',
3014:       2 => 'User is not performer error',
3015:     ),
3016:     'use_p' => 
3017:     array (
3018:       1 => 'Использовать',
3019:       2 => 'Use',
3020:     ),
3021:     'u_hash' => 
3022:     array (
3023:       1 => 'Хэш',
3024:       2 => 'U_hash',
3025:     ),
3026:     'very_expensive' => 
3027:     array (
3028:       1 => 'Слишком дорого',
3029:       2 => 'Very expensive',
3030:     ),
3031:     'view_p' => 
3032:     array (
3033:       1 => 'Посмотреть',
3034:     ),
3035:     'vote' => 
3036:     array (
3037:       1 => 'Голосовать',
3038:       2 => 'Vote',
3039:     ),
3040:     'voter' => 
3041:     array (
3042:       1 => 'Голосующий',
3043:       2 => 'Voter',
3044:     ),
3045:     'voting' => 
3046:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:11178`
```php
11166:       'about_ar' => '',
11167:       'about_fr' => '',
11168:       'about_es' => '',
11169:       'value_type' => '3',
11170:       'some' => '0',
11171:       'visibility' => '12',
11172:       'editable' => '0',
11173:     ),
11174:     3 => 
11175:     array (
11176:       'var' => 'session_token',
11177:       'ru' => 'Токен сессионного доступа',
11178:       'en' => 'Session access token',
11179:       'ar' => NULL,
11180:       'fr' => NULL,
11181:       'es' => NULL,
11182:       'about_ru' => '',
11183:       'about_en' => '',
11184:       'about_ar' => '',
11185:       'about_fr' => '',
11186:       'about_es' => '',
11187:       'value_type' => '1',
11188:       'some' => '0',
11189:       'visibility' => '12',
11190:       'editable' => '0',
11191:     ),
11192:     4 => 
11193:     array (
11194:       'var' => 'Phone',
11195:       'ru' => 'Телефон',
11196:       'en' => 'Phone',
11197:       'ar' => NULL,
11198:       'fr' => NULL,
11199:       'es' => NULL,
11200:       'about_ru' => '',
```

### `archive_17012026_1259/taxi/cache/data.php:15224`
```php
15212:         0 => '4',
15213:       ),
15214:       'roles_view' => 
15215:       array (
15216:         0 => '2',
15217:         1 => '4',
15218:       ),
15219:     ),
15220:     6 => 
15221:     array (
15222:       'var' => 'hold_s_token',
15223:       'ru' => 'Токен сессионного доступа юзера, запросившего холдирование',
15224:       'en' => 'Session access token of the user who requested the hold',
15225:       'ar' => NULL,
15226:       'fr' => NULL,
15227:       'es' => NULL,
15228:       'about_ru' => '',
15229:       'about_en' => '',
15230:       'about_ar' => '',
15231:       'about_fr' => '',
15232:       'about_es' => '',
15233:       'value_type' => '1',
15234:       'some' => '0',
15235:       'roles_edit' => 
15236:       array (
15237:         0 => '4',
15238:       ),
15239:       'roles_view' => 
15240:       array (
15241:         0 => '2',
15242:         1 => '4',
15243:       ),
15244:     ),
15245:     7 => 
15246:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:22848`
```php
22836:       'es' => NULL,
22837:       'about_ru' => '',
22838:       'about_en' => '',
22839:       'about_ar' => '',
22840:       'about_fr' => '',
22841:       'about_es' => '',
22842:       'value' => 'https://eitfa.ru/cgi/tg_app.py',
22843:     ),
22844:     'telegram_token' => 
22845:     array (
22846:       'group' => '17',
22847:       'ru' => 'Токен для доступа к серверу сервиса управления телеграмом',
22848:       'en' => 'Token for access to the telegram management service server',
22849:       'ar' => NULL,
22850:       'fr' => NULL,
22851:       'es' => NULL,
22852:       'about_ru' => '',
22853:       'about_en' => '',
22854:       'about_ar' => '',
22855:       'about_fr' => '',
22856:       'about_es' => '',
22857:       'value' => '',
22858:     ),
22859:     'stripe_webhook_signing_secret' => 
22860:     array (
22861:       'group' => '14',
22862:       'ru' => 'Секретный ключ страйпа для вебхука',
22863:       'en' => NULL,
22864:       'ar' => NULL,
22865:       'fr' => NULL,
22866:       'es' => NULL,
22867:       'about_ru' => '',
22868:       'about_en' => '',
22869:       'about_ar' => '',
22870:       'about_fr' => '',
```

### `archive_17012026_1259/taxi/controllers/c_google.php:24`
```php
12: if (!empty(taxi::$data_private['site_constants']['google_auth_client_id']['value'])) $clientID = trim(taxi::$data_private['site_constants']['google_auth_client_id']['value']);
13: if (!empty(taxi::$data_private['site_constants']['google_auth_client_secret']['value'])) $clientSecret = trim(taxi::$data_private['site_constants']['google_auth_client_secret']['value']);
14: if (empty($clientID) || empty($clientSecret)) header("Location: "."$taxiAppUrl?error=".urlencode('empty client_id or client_secret'));
15: 
16: $client = new Google_Client();
17: $client->setClientId($clientID);
18: $client->setClientSecret($clientSecret);
19: $client->setRedirectUri($redirectUri);
20: $client->addScope("email");
21: $client->addScope("profile");
22: 
23: if (isset($_GET['code'])) {
24: 	$token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
25: 	$client->setAccessToken($token['access_token']);
26: 
27: 	$google_oauth = new Google_Service_Oauth2($client);
28: 	$google_account_info = $google_oauth->userinfo->get();
29: 	$email =  $google_account_info->email;
30: 	$name =  $google_account_info->name;
31:  
32: 	$id = get_id_user("",$email,"");
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
```

### `archive_17012026_1259/taxi/controllers/c_google.php:25`
```php
13: if (!empty(taxi::$data_private['site_constants']['google_auth_client_secret']['value'])) $clientSecret = trim(taxi::$data_private['site_constants']['google_auth_client_secret']['value']);
14: if (empty($clientID) || empty($clientSecret)) header("Location: "."$taxiAppUrl?error=".urlencode('empty client_id or client_secret'));
15: 
16: $client = new Google_Client();
17: $client->setClientId($clientID);
18: $client->setClientSecret($clientSecret);
19: $client->setRedirectUri($redirectUri);
20: $client->addScope("email");
21: $client->addScope("profile");
22: 
23: if (isset($_GET['code'])) {
24: 	$token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
25: 	$client->setAccessToken($token['access_token']);
26: 
27: 	$google_oauth = new Google_Service_Oauth2($client);
28: 	$google_account_info = $google_oauth->userinfo->get();
29: 	$email =  $google_account_info->email;
30: 	$name =  $google_account_info->name;
31:  
32: 	$id = get_id_user("",$email,"");
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
```

### `archive_17012026_1259/taxi/controllers/c_index.php:187`
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
206: 
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:188`
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
207: 			Если type='e-mail_code' и password пустой, то
208: 				на емейл u_email, указанном в login, высылается четырехзначный код.
209: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
210: 			Если type='e-mail_code' и password указан, то	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2414`
```php
2402: 				longitude			долгота				необходимо
2403: 			Ответ сервера:
2404: 			{'code':'200','status':'success'}</pre>
2405: 		<fieldset class="form"><legend title="Записывает координаты авторизированного пользователя в базу. Доступна только для авторизованного пользователя.">Изменение координат пользователя</legend>
2406: 			<form class="complex" action="api/v1/location" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2415`
```php
2403: 			Ответ сервера:
2404: 			{'code':'200','status':'success'}</pre>
2405: 		<fieldset class="form"><legend title="Записывает координаты авторизированного пользователя в базу. Доступна только для авторизованного пользователя.">Изменение координат пользователя</legend>
2406: 			<form class="complex" action="api/v1/location" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
2437: 										"auth":			"число активных банов на авторизацию",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2423`
```php
2411: 			</form>
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
2437: 										"auth":			"число активных банов на авторизацию",
2438: 										"order":		"число активных банов на создание или получения поездки",
2439: 										"blog_topic":	"число активных банов на создание темы в блоге",
2440: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
2441: 										}
2442: 					"u_active":			"1 или 0",
2443: 					"u_photo":			"ссылка на фото",
2444: 					"u_birthday":		"день рождения пользователя в виде год-месяц-день",
2445: 					"u_lang":			"идентификатор языка, выбранного пользователем",	data.langs
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2424`
```php
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
2437: 										"auth":			"число активных банов на авторизацию",
2438: 										"order":		"число активных банов на создание или получения поездки",
2439: 										"blog_topic":	"число активных банов на создание темы в блоге",
2440: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
2441: 										}
2442: 					"u_active":			"1 или 0",
2443: 					"u_photo":			"ссылка на фото",
2444: 					"u_birthday":		"день рождения пользователя в виде год-месяц-день",
2445: 					"u_lang":			"идентификатор языка, выбранного пользователем",	data.langs
2446: 					"u_currency":		"iso4217 код валюты, выбранной пользователем",		data.currencies
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2451`
```php
2439: 										"blog_topic":	"число активных банов на создание темы в блоге",
2440: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
2441: 										}
2442: 					"u_active":			"1 или 0",
2443: 					"u_photo":			"ссылка на фото",
2444: 					"u_birthday":		"день рождения пользователя в виде год-месяц-день",
2445: 					"u_lang":			"идентификатор языка, выбранного пользователем",	data.langs
2446: 					"u_currency":		"iso4217 код валюты, выбранной пользователем",		data.currencies
2447: 					"u_gps_software":	"идентификатор навигации, выбранной пользователем	data.gps_softwares
2448: 				}
2449: 			}</pre>
2450: 		<fieldset class="form"><legend title="Значения токена и проверояного хеша авторизованного пользователя.">Получение токена и проверояного хеша</legend>
2451: 			<form action="api/v1/token" method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" data-method="get" target="_blank">
2452: 				<label class="no_border exclude only_post_request"><span>хеш авторизации</span><input data-name="auth_hash" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>
2453: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2454: 			</form>
2455: 		</fieldset>
2456: 		<pre>
2457: 	https://ibronevik.ru/taxi/api/v1/remind/								POST
2458: 		Восстановление пароля.
2459: 			Параметры запроса:
2460: 				u_email				емейл
2461: 				u_phone				телефон
2462: 				u_tg				идентификатор телеграм
2463: 				u_wa				идентификатор ватсап
2464: 			Ответ сервера:
2465: 			{'code':'200','status':'success'}</pre>
2466: 		<fieldset class="form"><legend title="Восстанавливает пароль: ищет пользователя по указанному емейлу, генерирует новый пароль и отправляет его на почту пользателя.">Восстановление пароля</legend>
2467: 			<form action="api/v1/remind/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2468: 				<label><span>Емейл</span><input name="u_email" type="text"></label>
2469: 				<label><span>Телефон</span><input name="u_phone" type="text"></label>
2470: 				<label><span>Идентификатор телеграм</span><input name="u_tg" type="text"></label>
2471: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2472: 			</form>
2473: 		</fieldset>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4078`
```php
4066: 												{"api_header":[[],"default_value_for_url"]}
4067: 												{"api_header":{"1":"default_value_for_url"}}
4068: 											Данные из указанных заголовков из апи передаются в url под новыми заголовками
4069: 											(при отсутствии передается дефолтное значение):
4070: 												{"api_header":["header_for_url","default_value_for_url"]}
4071: 											Значение ключа или нулевого элемента
4072: 												0		считается не указанным, загловки не меняются
4073: 												'0'		считается указанным, загловок меняется на '0'
4074: 												true	считается указанным, загловок меняется на '1'
4075: 												false	считается не указанным, загловки не меняются
4076: 					post_json				массив параметров с их значениями для запроса по url
4077: 											например:
4078: 												{"token":"long_string"}
4079: 					post_json_export		массив экспортируемых параметров, то есть:
4080: 												те параметры, которые будут браться из вызова апи
4081: 												и передаваться при запросе по ссылке url.
4082: 											Указанные параметры из апи передаются в url без изменений
4083: 											(при отсутствии не добавляются):
4084: 												{"api_request":null}
4085: 												{"api_request":""}
4086: 												{"api_request":[]}
4087: 												{"api_request":[null]}
4088: 												{"api_request":[""]}
4089: 												{"api_request":[[]]}
4090: 												{"api_request":{"3":"value"}}
4091: 												{"api_request":[null,null]}
4092: 												{"api_request":["",null]}
4093: 												{"api_request":[[],null]}
4094: 												{"api_request:{"2":null,"3":"value"}}
4095: 											Данные из указанных параметров из апи передаются в url под новыми параметрами
4096: 											(при отсутствии не добавляются):
4097: 												{"api_request":"request_for_url"}
4098: 												{"api_request":["request_for_url"]}
4099: 												{"api_request":[["request_for_url_key1","request_for_url_key2"]]}
4100: 											Указанные параметры из апи передаются в url без изменений
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4183`
```php
4171: 						"co_id1":{
4172: 							"co_id":		"co_id1 идентификатор контакта",
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
4204: 							?> 
4205: 						},	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4257`
```php
4245: 				Доступно только для авторизованного пользователя.
4246: 				Параметры запроса:				
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
4278: 						"co_id":				"идентификатор созданного контакта"
4279: 					}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4292`
```php
4280: 				}</pre>
4281: 		<fieldset class="form"><legend title="Создает контакт.">Создание контакта</legend>
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
4313: 				<input id="create_contact_data" name="data" type="hidden">
4314: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4333`
```php
4321: 				Параметры запроса:
4322: 					co_id		идентификатор контакта	 						необходимо
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
4354: 			<form id="edit_contact" class="complex" action="api/v1/contact/edit/" data-action="api/v1/contact/edit/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
4355: 				<label><span>идентификатор контакта</span><input data-name="co_id" name="co_id" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4365`
```php
4353: 		<fieldset class="form"><legend title="Редактирует контакт.">Редактирование контакта</legend>
4354: 			<form id="edit_contact" class="complex" action="api/v1/contact/edit/" data-action="api/v1/contact/edit/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
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
4386: 				<input id="edit_contact_data" name="data" type="hidden">
4387: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5574`
```php
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5575`
```php
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5580`
```php
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5581`
```php
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
5603: 			cache version
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5587`
```php
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
5603: 			cache version
5604: 		А значение, соответствующее этому ключу, будет версией.
5605: 	
5606: 		https://ibronevik.ru/taxi/api/v1/drive							POST
5607: 			При создание поездки 'Вызов на дороге' в ответе сервера выводится "b_driver_code".
5608: 			Идентификатор класса машины	теперь не обязателен (в этом случае считается null).
5609: 				null означает, что класс машины любой.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5588`
```php
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
5603: 			cache version
5604: 		А значение, соответствующее этому ключу, будет версией.
5605: 	
5606: 		https://ibronevik.ru/taxi/api/v1/drive							POST
5607: 			При создание поездки 'Вызов на дороге' в ответе сервера выводится "b_driver_code".
5608: 			Идентификатор класса машины	теперь не обязателен (в этом случае считается null).
5609: 				null означает, что класс машины любой.
5610: 			Для цели поезки теперь не обязательно указывать или адрес или координаты.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5616`
```php
5604: 		А значение, соответствующее этому ключу, будет версией.
5605: 	
5606: 		https://ibronevik.ru/taxi/api/v1/drive							POST
5607: 			При создание поездки 'Вызов на дороге' в ответе сервера выводится "b_driver_code".
5608: 			Идентификатор класса машины	теперь не обязателен (в этом случае считается null).
5609: 				null означает, что класс машины любой.
5610: 			Для цели поезки теперь не обязательно указывать или адрес или координаты.
5611: 			Можно указывать дополнительные параметры b_options.
5612: 
5613: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}				POST
5614: 			Добавлено изменение максимального времени ожидания set_waiting_time.
5615: 			
5616: 		https://ibronevik.ru/taxi/api/v1/token							GET
5617: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5618: 			Добавлено подтверждение авторизации через auth_hash параметр.
5619: 
5620: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
5621: 			Добавлен вывод auth_hash.
5622: 
5623: 		Приоритет методов авторизации:
5624: 			auth_token и auth_u_hash
5625: 			auth_hash
5626: 			куки сессии
5627: 
5628: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5629: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5630: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5631: 			Список отображаемых поездок теперь зависит от b_start_datetime+b_max_waiting.
5632: 
5633: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5634: 			В данные о поездках добавлено b_max_waiting_list.
5635: 			
5636: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5637: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5638: 		https://ibronevik.ru/taxi/api/v1/drive/archive					GET
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5617`
```php
5605: 	
5606: 		https://ibronevik.ru/taxi/api/v1/drive							POST
5607: 			При создание поездки 'Вызов на дороге' в ответе сервера выводится "b_driver_code".
5608: 			Идентификатор класса машины	теперь не обязателен (в этом случае считается null).
5609: 				null означает, что класс машины любой.
5610: 			Для цели поезки теперь не обязательно указывать или адрес или координаты.
5611: 			Можно указывать дополнительные параметры b_options.
5612: 
5613: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}				POST
5614: 			Добавлено изменение максимального времени ожидания set_waiting_time.
5615: 			
5616: 		https://ibronevik.ru/taxi/api/v1/token							GET
5617: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5618: 			Добавлено подтверждение авторизации через auth_hash параметр.
5619: 
5620: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
5621: 			Добавлен вывод auth_hash.
5622: 
5623: 		Приоритет методов авторизации:
5624: 			auth_token и auth_u_hash
5625: 			auth_hash
5626: 			куки сессии
5627: 
5628: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5629: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5630: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5631: 			Список отображаемых поездок теперь зависит от b_start_datetime+b_max_waiting.
5632: 
5633: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5634: 			В данные о поездках добавлено b_max_waiting_list.
5635: 			
5636: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5637: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5638: 		https://ibronevik.ru/taxi/api/v1/drive/archive					GET
5639: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6066`
```php
6054: 		Для неавторизованного пользователя параметр игнорируется. 
6055: 		Стандартный auth_user это данные, которые добавляются к информации основного запроса.
6056: 		Разсширенный auth_user это данные, которые выводятся при авторизации.
6057: 		Полный auth_user это данные, аналогичные taxi/api/v1/user/authorized.
6058: 		Разсширенный auth и полный auth может иметь вид:
6059: 			'error': 'описание ошибки'
6060: 		Если au=e, то к любому ответу сервера будет добавлятся расширенный 'auth_user'.
6061: 		Если au=f, то к любому ответу сервера будет добавлятся полный 'auth_user' с полным 'auth_user'.
6062: 
6063: 		В стандартный массив данный	'auth_user' добавлено:		
6064: 			u_lang
6065: 
6066: 		Для неправильных значений token или u_hash к ответу сервера добавляется auth_error.
6067: 
6068: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}
6069: 		https://ibronevik.ru/taxi/api/v1/user
6070: 			Водитель с u_check_state=2 может редактировать: 
6071: 				"u_active"
6072: 
6073: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
6074: 			В вывод файлов добавлены массивы:
6075: 				"cargo_case_models":{...}
6076: 				"cargo_case_types":{...}
6077: 				"cargo_value_types":{...}				
6078: 				"units_of_length":{...},
6079: 				"units_of_mass":{...},
6080: 				"units_of_volume":{...},	
6081: 				"unit_sets":{...},
6082: 		
6083: 		Описание изменений апи, которые будут добавлены для грузоперевозок:
6084: 			Для пользователя u_active может принимать не только 0 и 1, но и 2.
6085: 				0 				не работает
6086: 				1				свободен
6087: 				2				выполняет заказ, но готов подвезти
6088: 				3				выполняет заказ и не принимает новые
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6654`
```php
6642: 			<select id="type_for_get_fields_par"><option value="activeOrder" selected>Активные поездки</option><option value="processingOrder">Поездки, ожидающие одобрения</option><option value="archiveOrder">Архивные поездки</option><option value="order">Определенные поездки</option></select> <select id="role_for_get_fields_par"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option></select> <input id="str_arr_for_get_fields_par" value="[&quot;b_offers&quot;,&quot;b_offer&quot;]"> <input id="button_for_get_fields_par" type="button" value="получить параметр fields"> <span id="res_for_get_fields_par"></span>
6643: 			<script>
6644: 				button_for_get_fields_par.onclick = function(){
6645: 					res_for_get_fields_par.textContent = get_fields_par(JSON.parse(str_arr_for_get_fields_par.value),type_for_get_fields_par.value,role_for_get_fields_par.value);
6646: 				};
6647: 				button_for_get_fields_par.click();
6648: 
6649: 			</script>
6650: 		</pre>	
6651: 		<pre>
6652: 	Обновление (июнь 2022)
6653: 		https://ibronevik.ru/taxi/api/v1/push/{p_id}/add				POST
6654: 			c POST параметрами u_id, "token=test token", "u_hash=test u_hash"
6655: 			Заглушка для метода присвоения пользователю идентификатора сервиса сообщений
6656: 		
6657: 		https://ibronevik.ru/taxi/api/v1/register/						POST
6658: 			Добавлен параметр:
6659: 				data.u_details	
6660: 		</pre>
6661: 		<pre>
6662: 	Обновление (ноябрь 2022)
6663: 		https://ibronevik.ru/taxi/api/v1/register/						POST
6664: 			Добавлен параметр:
6665: 				st
6666: 				
6667: 		Администратор может заходить как любой пользователь с помощью POST параметра:
6668: 				u_a_id			идентификатор пользователя
6669: 				u_a_email		емейл пользователя
6670: 				u_a_phone		телефон пользователя
6671: 		Приоритет использования этих параметров для авторизации:
6672: 				u_a_id
6673: 				u_a_email
6674: 				u_a_phone
6675: 				
6676: 		Таким образом на сервере приложения для авторизации через гугл предлагается следующая схема:
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6681`
```php
6669: 				u_a_email		емейл пользователя
6670: 				u_a_phone		телефон пользователя
6671: 		Приоритет использования этих параметров для авторизации:
6672: 				u_a_id
6673: 				u_a_email
6674: 				u_a_phone
6675: 				
6676: 		Таким образом на сервере приложения для авторизации через гугл предлагается следующая схема:
6677: 			Создается пользователь админ, допустим, "веб приложение такси".
6678: 			Пользователь "веб приложение такси" получает токен, который сохраняется на сервере приложения.
6679: 			При авторизации через гугл получаем емейл пользователя.
6680: 			На сервере приложения "веб приложение такси" отправляет запрос:
6681: 				POST, https://ibronevik.ru/taxi/api/v1/token/authorized с данными: u_a_email="полученный емейл"
6682: 			Из ответа получаем токен.
6683: 			Если ответ содержит ошибку:
6684: 				{'code' => '404', 'status' => 'error', 'message' => 'user not found'}
6685: 				то приложению пользователя сообщается, что требуется регистрация.
6686: 				Далее приложение пользователя регистрирует его
6687: 					POST, https://ibronevik.ru/taxi/api/v1/register/ с данными: u_email="полученный емейл"&st
6688: 				токен из ответа используется приложением пользователя.
6689: 		</pre>
6690: 		<pre>
6691: 	Обновление (январь 2023)
6692: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
6693: 			Добавлен type:
6694: 				whatsapp
6695: 			
6696: 		https://ibronevik.ru/taxi/google/
6697: 			Добавлена страница редиректа для аторизации через гугл.
6698: 			Направляет на гугл авторизацию.
6699: 			При возврате на нее с гугл:
6700: 				пользователь с емейлом гугл не зарегистрирован:
6701: 					редирект на приложение такси с GET параметрами u_email и u_name
6702: 				в случае ошибок:
6703: 					с GET параметром error
```

### `archive_17012026_1259/taxi/controllers/c_index.php:7830`
```php
7818: 	Обновление 2 (ноябрь 2023)	
7819: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
7820: 			В основной блок добавлены массивы:
7821: 				"script_templates":{...}
7822: 			Добавлена обработка вывода script_templates.id.file
7823: 
7824: 		https://ibronevik.ru/taxi/api/v1/script/template/{script_id}				POST	
7825: 		https://ibronevik.ru/taxi/api/v1/script/template/{script_var}?is_var=1		POST
7826: 			Добавлен методы выполнения скрипта	
7827: 		</pre>
7828: 		<pre>
7829: 	Обновление 3 (ноябрь 2023)
7830: 		Поправлена проверка u_hash, когда вводится некорректное значение
7831: 
7832: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
7833: 			В выводе блока sс изменены массивы:
7834: 				"schedule":{...}
7835: 					добавлено:
7836: 						options
7837: 			В основном блоке изменены массивы:
7838: 				"langs":{...}
7839: 					добавлено:
7840: 						tr_code
7841: 	
7842: 		Для редактирования языковых слов добавлены:
7843: 			regions
7844: 			script_templates
7845: 			site_images
7846: 			sql_templates
7847: 			stadiums
7848: 			teams
7849: 			tournaments
7850: 		
7851: 		Также добавлен автоматический перевод
7852: 			
```

### `archive_17012026_1259/taxi/controllers/c_index.php:8921`
```php
8909: 		</pre>
8910: 		<pre>
8911: 	Обновление 3 (июнь 2024)
8912: 		https://ibronevik.ru/taxi/api/v1/dropbox/file/{dl_id}
8913: 			Добавлено имя файла в content-disposition
8914: 
8915: 		Для указания адреса веб приложения для различных редиректов добавлен GET или POST параметр:
8916: 			appUrl
8917: 		</pre>
8918: 		<pre>
8919: 	Обновление 4 (июнь 2024)
8920: 		https://ibronevik.ru/taxi/api/v1/dropbox/file/{dl_id}
8921: 			Добавлен Access-Control-Expose-Headers: Content-disposition
8922: 
8923: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
8924: 			В выводе блока sс изменены массивы:
8925: 				"schedule":{...}
8926: 					добавлено:
8927: 						currency
8928: 						currency_priority
8929: 	
8930: 		https://ibronevik.ru/taxi/api/v1/trip							POST	
8931: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}				POST
8932: 			Добавлен параметр
8933: 				currency
8934: 				currency_priority
8935: 
8936: 		https://ibronevik.ru/taxi/api/v1/trip/							GET
8937: 		https://ibronevik.ru/taxi/api/v1/trip/now						GET
8938: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id1,t_id5}			GET
8939: 			Добавлено
8940: 				currency
8941: 				currency_priority
8942: 				sc_currency
8943: 				sc_currency_priority
```

### `archive_17012026_1259/taxi/controllers/c_index.php:9367`
```php
9355: 			Добавлено для nocache:
9356: 				top
9357: 				options
9358: 				time_zone
9359: 				price_time_function
9360: 				currency
9361: 				currency_priority
9362: 				fee
9363: 				tariff
9364: 				tariff_priority
9365: 				code_ean_base64
9366: 
9367: 		Добавлена правильная обработка неправильных коротких u_hash и auth_hash
9368: 	
9369: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/edit/	POST			
9370: 			Для параметра pass добавлено логирование действий билетера
9371: 
9372: 		https://ibronevik.ru/taxi/api/v1/ticket/check/log					GET
9373: 			Добавлен метод вывода логов.
9374: 								
9375: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
9376: 			В основной блок добавлены массивы:
9377: 				"map_place_polygons":{...}
9378: 				"favorite_addresses":{...}
9379: 			Ключи для каждого объекта map_place_polygons:
9380: 				upper
9381: 				var
9382: 				coordinates
9383: 				<?php
9384: 					$sql_name = $sql_description = array();			
9385: 					foreach (taxi::$data['langs'] as $lang)
9386: 					{
9387: 						$sql_name[] = $lang['iso'];
9388: 						$sql_description[] = "about_{$lang['iso']}";
9389: 					}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:10965`
```php
10953: 				auth.onsubmit = function(){
10954: 			        auth_type_data.value = auth_type.value + (auth_type_add.value ? ',' + auth_type_add.value : '');
10955: 				};   
10956: 
10957: 				document.getElementsByTagName('form').forEach(function(el){
10958: 					el.setAttribute("data-method",el.method);
10959: 					var label = '<label class="no_border"><span>Показывать версию data.js</span><input class="checkbox" name="cv" type="checkbox" value=""></label>';
10960: 					label += '<label class="no_border"><span>Включить дебаг</span><input class="checkbox" name="debug" type="checkbox" value=""></label>';
10961: 					label += '<label class="no_border exclude"><span>Показывать данные авторизованного пользователя</span><select data-name="au"><option value="" selected>Стандартный auth_user</option><option value="e">Расширенный auth_user</option><option value="f">Полный auth_user</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10962: 					if (el.classList.contains("type_select")) {
10963: 						label += '<label class="no_border exclude"><span>формат массива</span><input data-name="array_type" type="text" value="list"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10964: 					}
10965: 					label += '<label class="no_border exclude only_post_request"><span>токен</span><input data-name="token" type="text" value="' + auth_token.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10966: 					label += '<label class="no_border exclude only_post_request"><span>проверочный хеш</span><input data-name="u_hash" type="text" value="' + auth_u_hash.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10967: 					if (el.classList.contains("out_control")) {
10968: 						label += '<label class="no_border exclude"><span>номер первой выводимой записи (начиная с нуля)</span><input data-name="lo" type="text" value="0"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10969: 						label += '<label class="no_border exclude"><span>количество выводимых записей</span><input data-name="lc" type="text" value="5"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10970: 					}
10971: 					label += '<label class="no_border exclude"><span>идентификатор языка</span><input data-name="lang" type="text" value="1"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10972: 					label += '<label class="no_border exclude"><span>роль пользователя</span><select data-name="u_a_role"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option><option value="6">Билетер</option><option value="7">Билетер с расширенными полномочиями</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10973: 					label += '<label class="no_border exclude only_post_request"><span>идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_id" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10974: 					label += '<label class="no_border exclude only_post_request"><span>емейл пользователя, авторизация которого имитируется</span><input data-name="u_a_email" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10975: 					label += '<label class="no_border exclude only_post_request"><span>телефон пользователя, авторизация которого имитируется</span><input data-name="u_a_phone" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10976: 					label += '<label class="no_border exclude only_post_request"><span>телеграм идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_tg" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10977: 					label += '<label class="no_border exclude only_post_request"><span>ватсап идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_wa" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10978: 					label += '<label class="no_border exclude"><span>адрес веб приложения</span><input data-name="appUrl" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10979: 					label += '<label class="no_border exclude"><span>токен сессионного доступа</span><input data-name="s_token" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10980: 					el.insertAdjacentHTML("AfterBegin",label);
10981: 				});
10982: 
10983: 				(function(){
10984: 					var input = get_version.querySelector('input[type="checkbox"]');
10985: 					input.checked = true;
10986: 					var label = input.parentNode;
10987: 					label.style.display = "none";
```

### `archive_17012026_1259/taxi/controllers/c_index.php:10966`
```php
10954: 			        auth_type_data.value = auth_type.value + (auth_type_add.value ? ',' + auth_type_add.value : '');
10955: 				};   
10956: 
10957: 				document.getElementsByTagName('form').forEach(function(el){
10958: 					el.setAttribute("data-method",el.method);
10959: 					var label = '<label class="no_border"><span>Показывать версию data.js</span><input class="checkbox" name="cv" type="checkbox" value=""></label>';
10960: 					label += '<label class="no_border"><span>Включить дебаг</span><input class="checkbox" name="debug" type="checkbox" value=""></label>';
10961: 					label += '<label class="no_border exclude"><span>Показывать данные авторизованного пользователя</span><select data-name="au"><option value="" selected>Стандартный auth_user</option><option value="e">Расширенный auth_user</option><option value="f">Полный auth_user</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10962: 					if (el.classList.contains("type_select")) {
10963: 						label += '<label class="no_border exclude"><span>формат массива</span><input data-name="array_type" type="text" value="list"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10964: 					}
10965: 					label += '<label class="no_border exclude only_post_request"><span>токен</span><input data-name="token" type="text" value="' + auth_token.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10966: 					label += '<label class="no_border exclude only_post_request"><span>проверочный хеш</span><input data-name="u_hash" type="text" value="' + auth_u_hash.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10967: 					if (el.classList.contains("out_control")) {
10968: 						label += '<label class="no_border exclude"><span>номер первой выводимой записи (начиная с нуля)</span><input data-name="lo" type="text" value="0"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10969: 						label += '<label class="no_border exclude"><span>количество выводимых записей</span><input data-name="lc" type="text" value="5"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10970: 					}
10971: 					label += '<label class="no_border exclude"><span>идентификатор языка</span><input data-name="lang" type="text" value="1"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10972: 					label += '<label class="no_border exclude"><span>роль пользователя</span><select data-name="u_a_role"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option><option value="6">Билетер</option><option value="7">Билетер с расширенными полномочиями</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10973: 					label += '<label class="no_border exclude only_post_request"><span>идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_id" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10974: 					label += '<label class="no_border exclude only_post_request"><span>емейл пользователя, авторизация которого имитируется</span><input data-name="u_a_email" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10975: 					label += '<label class="no_border exclude only_post_request"><span>телефон пользователя, авторизация которого имитируется</span><input data-name="u_a_phone" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10976: 					label += '<label class="no_border exclude only_post_request"><span>телеграм идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_tg" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10977: 					label += '<label class="no_border exclude only_post_request"><span>ватсап идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_wa" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10978: 					label += '<label class="no_border exclude"><span>адрес веб приложения</span><input data-name="appUrl" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10979: 					label += '<label class="no_border exclude"><span>токен сессионного доступа</span><input data-name="s_token" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10980: 					el.insertAdjacentHTML("AfterBegin",label);
10981: 				});
10982: 
10983: 				(function(){
10984: 					var input = get_version.querySelector('input[type="checkbox"]');
10985: 					input.checked = true;
10986: 					var label = input.parentNode;
10987: 					label.style.display = "none";
10988: 				})();
```

### `archive_17012026_1259/taxi/controllers/c_stripe.php:63`
```php
51: 		if ($hash != $key) json_exit('400', 'error', 'wrong tilda form webhook key');
52: 		$status = 'succeeded';
53: 	}
54: 	elseif (isset($api_use) && $api_use === true)
55: 	{
56: 		if (is_array($order)) {
57: 			$order_arr = $order;
58: 			$i = 0;
59: 		}
60: 	}
61: 	else
62: 	{
63: 		json_exit('400', 'error', 'wrong request');
64: 	}
65: 	while (true)
66: 	{
67: 		if (isset($order_arr))
68: 		{
69: 			if (!isset($order_arr[$i])) break;
70: 			$order = $order_arr[$i];
71: 			$i++;
72: 		}
73: 		$error = array();
74: 		if (empty($error) && DEFAULT_PROFILE == 'stadium')
75: 		{
76: 			switch ($status){
77: 				case 'pending':		
78: 					break;
79: 				case 'succeeded':
80: 				case 'failed':
81: 				case 'processing':
82: 					$s = "SELECT
83: 							`id_order`,
84: 							`client`,
85: 							`options`,
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:304`
```php
292: 						$out = $API->selectTrip(NULL,'processing',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
293: 					}
294: 				}
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
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
317: 					if ($_GET['par'][3] == 'send')
318: 					{
319: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
320: 					}
321: 				}
322: 			}
323: 			elseif ($_GET['par'][1] == 'referral')
324: 			{
325: 				if (!empty($_GET['par'][2]))
326: 				{
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:309`
```php
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
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
317: 					if ($_GET['par'][3] == 'send')
318: 					{
319: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
320: 					}
321: 				}
322: 			}
323: 			elseif ($_GET['par'][1] == 'referral')
324: 			{
325: 				if (!empty($_GET['par'][2]))
326: 				{
327: 					if ($_GET['par'][2] == 'code')
328: 					{
329: 						if (!empty($_GET['par'][3]))
330: 						{
331: 							if ($_GET['par'][4] == 'check')
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:595`
```php
583: 						$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
584: 					}
585: 					else
586: 					{
587: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
588: 					}
589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
590: 				}
591: 			}
592: 		}
593: 		if (isset($_SESSION['token_auth']) && empty($_SESSION[UID])) 
594: 		{
595: 			$out['auth_error'] = 'wrong token or u_hash';
596: 		}
597: 	}
598: 
599: 	header('Content-Type: application/json;charset=UTF-8');
600: 	echo json_encode($out);
601: 	die();
602: ?>
```

### `archive_17012026_1259/taxi/controllers/c_api.php:312`
```php
300: 						$out = $API->selectTrip(NULL,'processing',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0,isset($_REQUEST['raw_price'])?true:false,isset($_REQUEST['wi'])?true:false,isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array(),isset(taxi::$data_private['aggregators'])?taxi::$data_private['aggregators']:array());
301: 					}
302: 				}
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
317: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
318: 
319: 				}
320: 			}
321: 			elseif ($_GET['par'][1] == 'mail')
322: 			{
323: 				if (!empty($_GET['par'][3]))
324: 				{
325: 					if ($_GET['par'][3] == 'send')
326: 					{
327: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
328: 					}
329: 				}
330: 			}
331: 			elseif ($_GET['par'][1] == 'referral')
332: 			{
333: 				if (!empty($_GET['par'][2]))
334: 				{
```

### `archive_17012026_1259/taxi/controllers/c_api.php:317`
```php
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
317: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
318: 
319: 				}
320: 			}
321: 			elseif ($_GET['par'][1] == 'mail')
322: 			{
323: 				if (!empty($_GET['par'][3]))
324: 				{
325: 					if ($_GET['par'][3] == 'send')
326: 					{
327: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
328: 					}
329: 				}
330: 			}
331: 			elseif ($_GET['par'][1] == 'referral')
332: 			{
333: 				if (!empty($_GET['par'][2]))
334: 				{
335: 					if ($_GET['par'][2] == 'code')
336: 					{
337: 						if (!empty($_GET['par'][3]))
338: 						{
339: 							if ($_GET['par'][4] == 'check')
```

### `archive_17012026_1259/taxi/controllers/c_api.php:855`
```php
843: 						$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
844: 					}
845: 					else
846: 					{
847: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
848: 					}
849: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
850: 				}
851: 			}
852: 		}
853: 		if (isset($_SESSION['token_auth']) && empty($_SESSION[UID])) 
854: 		{
855: 			$out['auth_error'] = 'wrong token or u_hash';
856: 		}
857: 	}
858: 
859: 	header('Content-Type: application/json;charset=UTF-8');
860: 	echo json_encode($out);
861: 	die();
862: ?>
```

## 6. Правило финального Claim

`CONFIRMED` допускается только если можно показать:

```text
Frontend:
credential value/field
        ↓
apiMethod()
        ↓
HTTP transport
```

и:

```text
Backend:
HTTP input
        ↓
credential extraction
        ↓
check_auth_user()
        ↓
authenticated User
```

Само совпадение названия `token`, `u_hash` или `Authorization` недостаточно.

## 7. Current status

После RP-37 v0.3 мы имеем детальную карту обеих сторон value-flow.

Если конкретный field/value совпадает между `apiMethod()` и `check_auth_user()`, Authentication bridge можно повысить до:

```text
CONFIRMED
```

Если frontend credential передаётся одним способом, а backend ожидает другой и преобразование не найдено:

```text
UNKNOWN / VALUE_FLOW_UNRESOLVED
```

Если код прямо показывает, что credential не используется backend auth gate:

```text
REJECTED
```

## 8. Следующий шаг

Не создавать новую вертикаль.

Нужно сделать **один ручной semantic join** по конкретному credential field:

```text
Frontend apiMethod field
        ↕
HTTP transport
        ↕
Backend request field
        ↕
check_auth_user()
```

После этого либо закрываем Authentication, либо фиксируем конкретный `Research Question` для оставшегося gap.
