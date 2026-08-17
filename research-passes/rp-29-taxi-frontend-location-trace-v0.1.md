# Backend Semantic Graph — Research Pass 29
# Taxi Frontend Location / Assigned Driver Trace v0.1

**Статус:** PARTIALLY CONFIRMED / FRONTEND SOURCE FOUND  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-28 Frontend Source Gap Closure v0.1  
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

## 1. Почему RP-29 меняет результат RP-28

RP-28 исследовал старый backend snapshot, где полноценного Taxi Web frontend source не было.

Теперь предоставлен отдельный versioned artifact:

```text
taxi-master.zip
```

Это полноценный React/TypeScript frontend source.

Поэтому `Taxi Web → SOURCE_GAP` из RP-28 для этого нового snapshot больше не действует. RP-28 остаётся корректным для своего source corpus.

## 2. Frontend location pipeline

Есть отдельный API adapter:

```text
src/API/location.ts
```

Он делает:

```text
sendPosition(latitude, longitude)
    ↓
POST {API_URL}/location
```

`apiMethod()` автоматически добавляет authentication fields:

```text
token
u_hash
```

### Evidence

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

Это подтверждает frontend authenticated self-location write path.

## 3. Geolocation state и отправка позиции

Frontend содержит Redux geolocation module.

Основной поток:

```text
navigator.geolocation
    ↓
getCurrentPosition()
    ↓
GET_SUCCESS
    ↓
geolocation state
```

При активном sending:

```text
geoposition state
    ↓
sendPosition()
    ↓
POST /location
```

### Evidence

```text
1: import { channel } from 'redux-saga'
2: import { all, race, take, takeEvery, put, delay } from 'redux-saga/effects'
3: import { TAction } from '../../types'
4: import { getCurrentPosition } from '../../tools/utils'
5: import { select, call, whileWatching } from '../../tools/sagaUtils'
6: import { sendPosition } from '../../API/location'
7: import { ActionTypes } from './constants'
8: import { geoposition as geopositionSelector } from './selectors'
9: 
10: export function* saga() {
11:   yield all([
12:     call(geolocationSaga),
13:   ])
14: }
15: 
16: function* geolocationSaga() {
17:   let latestGet = 0
18:   let latestSent: GeolocationPosition | undefined
19:   const listeners = new Map<number, number>()
20:   let pollInterval = Infinity
21:   const intervalChangeChannel = yield* call(channel)
22: 
23:   yield all([
24:     call(function*() {
25:       while (true) {
26:         if (pollInterval === Infinity) {
27:           yield take(intervalChangeChannel)
28:           continue
29:         }
30:         const [timePassed] = yield race([
31:           delay(Math.max(latestGet + pollInterval - Date.now(), 0), true),
32:           take(intervalChangeChannel),
33:         ])
34:         if (!timePassed)
35:           continue
36: 
37:         yield* getGeopositionSaga()
38:         latestGet = Date.now()
39:       }
40:     }),
41: 
42:     takeEvery(ActionTypes.WATCH, function*({ payload: { interval } }: TAction) {
43:       listeners.set(interval, (listeners.get(interval) ?? 0) + 1)
44:       if (interval < pollInterval) {
45:         pollInterval = interval
46:         yield put(intervalChangeChannel, {})
47:       }
48:     }),
49: 
50:     takeEvery(ActionTypes.UNWATCH, function*({
51:       payload: { interval },
52:     }: TAction) {
53:       const listenersWithInterval = listeners.get(interval)!
54:       if (listenersWithInterval > 1)
55:         listeners.set(interval, listenersWithInterval - 1)
56:       else {
57:         listeners.delete(interval)
58:         if (interval === pollInterval) {
59:           pollInterval = [...listeners.keys()].reduce(
60:             (min, interval) => interval < min ? interval : min,
61:             Infinity,
62:           )
63:           yield put(intervalChangeChannel, {})
64:         }
65:       }
66:     }),
67: 
68:     whileWatching(
69:       ActionTypes.ACTIVATE_SENDING,
70:       ActionTypes.DEACTIVATE_SENDING,
71: 
72:       function*() {
73:         latestSent = yield* sendPositionSaga(latestSent)
74:         yield take(ActionTypes.GET_SUCCESS)
75:       },
76:     ),
77:   ])
78: }
79: 
80: function* getGeopositionSaga() {
81:   try {
82:     const geoposition = yield* call(getCurrentPosition)
83:     yield put({ type: ActionTypes.GET_SUCCESS, payload: geoposition })
84:   } catch (error) {
85:     yield put({ type: ActionTypes.GET_FAIL, payload: error })
86:   }
87: }
88: 
89: function* sendPositionSaga(latestSent?: GeolocationPosition) {
90:   const geoposition = yield* select(geopositionSelector)
```

Однако в source найден `ACTIVATE_SENDING`, а production invocation `activateSending()` не найден.

Поэтому нельзя утверждать:

```text
Driver page always sends current position
```

Статус фактической runtime activation:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

## 4. Watch geolocation — не то же самое, что sending

`watchReadyOrders()` запускает:

```text
watchGeolocation({ interval: 1 hour })
```

### Evidence

```text
1: import { ParametersExceptFirst, TAction } from '../../types'
2: import { IOrder } from '../../types/types'
3: import { IResponse } from '../../types/api'
4: import { candidateMode }  from '../../tools/order'
5: import * as API from '../../API'
6: import { IRootState, IDispatch } from '..'
7: import { watch as watchGeolocation } from '../geolocation/actionCreators'
8: import { ActionTypes } from './constants'
9: import { order as orderSelector } from './selectors'
10: 
11: const READY_ORDERS_GEOLOCATION_INTERVAL = 1000 * 60 * 60
12: 
13: export const watchActiveOrders = () => (dispatch: IDispatch) => {
14:   dispatch({ type: ActionTypes.WATCH_ACTIVE_ORDERS })
15:   return () => {
16:     dispatch({ type: ActionTypes.UNWATCH_ACTIVE_ORDERS })
17:   }
18: }
19: export const watchReadyOrders = () => (dispatch: IDispatch) => {
20:   const unwatch = dispatch(watchGeolocation({
21:     interval: READY_ORDERS_GEOLOCATION_INTERVAL,
22:   }))
23:   dispatch({ type: ActionTypes.WATCH_READY_ORDERS })
24:   return () => {
25:     dispatch({ type: ActionTypes.UNWATCH_READY_ORDERS })
26:     unwatch()
27:   }
28: }
29: export const watchHistoryOrders = () => (dispatch: IDispatch) => {
30:   dispatch({ type: ActionTypes.WATCH_HISTORY_ORDERS })
31:   return () => {
32:     dispatch({ type: ActionTypes.UNWATCH_HISTORY_ORDERS })
33:   }
34: }
35: 
```

Но это только получение geolocation.

В `geolocation/sagas.ts` отправка на `/location` активируется отдельным `ACTIVATE_SENDING`.

Следовательно:

```text
WATCH_GEOLOCATION
    ≠
ACTIVATE_GEOLOCATION_SENDING
```

## 5. Ключевая находка: Order содержит координаты Driver

Frontend `IDriver` содержит:

```text
u_id
c_latitude
c_longitude
l_datetime
```

Комментарии прямо определяют:

```text
c_latitude  = Широта водителя
c_longitude = Долгота водителя
l_datetime  = Дата получения координат
```

### Evidence

```text
145:   id: string
146:   type: EBookingCommentTypes
147:   responseMode?: EDriverResponseModes
148:   internal?: boolean
149: }
150: 
151: export interface IDriver {
152:   /** Идентификатор водителя */
153:   u_id: string,
154:   /** Идентификатор машины */
155:   c_id: string,
156:   /** На месте ли машина */
157:   c_arrive_state: boolean,
158:   /** Идентификатор статуса водителя, data.booking_driver_states */
159:   c_state: EBookingDriverState,
160:   /** Широта водителя */
161:   c_latitude?: number,
162:   /** Долгота водителя */
163:   c_longitude?: number,
164:   /** Дата получения координат */
165:   l_datetime?: Moment,
166:   /** идентификатор способа оплаты */
167:   c_payment_way?: EPaymentWays,
168:   /** идентификатор платежной карты */
169:   c_payment_card?: string,
170:   /** сумма оплаты водителем за сервис сайта */
171:   c_payment_sum?: number
172:   /** дата оплаты */
173:   c_payment_datetime?: Moment,
174:   /** причина отмены поездки водителем */
175:   c_cancel_reason?: string,
176:   /** оценка поездки */
177:   c_rating?: string,
178:   /** дата подачи заявки на исполнение */
179:   c_becomed_candidate?: Moment,
180:   /** дата отмены исполнения */
181:   c_canceled?: Moment,
182:   /** дата прибытия машины */
183:   c_arrived?: Moment,
184:   /** Дата получения координат */
185:   c_started?: Moment,
186:   /** дата начала поездки */
187:   c_completed?: Moment,
188:   /** дата завершения поезки */
189:   c_tips?: number
190:   c_options?: ICarOptions
191: }
192: 
193: export interface IOptions {
194:   submitPrice?: string;
195:   fromShortAddress?: string
196:   toShortAddress?: string
197:   courier_auto?: string | ECourierAutoTypes
198:   from_porch?: string
199:   from_floor?: string
200:   from_room?: string
201:   from_way?: string
202:   from_mission?: string
203:   from_tel?: string | null
204:   from_day?: string
205:   from_time_from?: string | null
206:   from_time_to?: string | null
207:   to_porch?: string
208:   to_floor?: string
209:   to_room?: string
210:   to_way?: string
211:   to_mission?: string
212:   to_tel?: string | null
213:   to_day?: string
214:   to_time_from?: string | null
215:   to_time_to?: string | null
216:   object?: string
217:   weight?: string
218:   is_big_size?: boolean
219:   cost?: number
220:   is_loading_needs?: boolean
221:   customer_price?: number
222:   moveType?: EMoveTypes
223:   steps?: string
224:   elevator?: IElevatorState
225:   furniture?: IFurniture['house'] | IFurniture['room']
226:   time_is_not_important?: boolean
227:   fromDateTimeInterval?: IDateTime
228:   tillDateTimeInterval?: IDateTime
229:   bigTruckCargo?: string
230:   size?: number
```

Следовательно, frontend data model уже предполагает:

```text
Order
  └── drivers[]
        ├── u_id
        ├── c_latitude
        ├── c_longitude
        └── l_datetime
```

## 6. Backend → Frontend order bridge

`getOrders()` получает `response.data.booking`, после чего вызывает `convertOrder(item)`.

`convertOrder()` вызывает `convertDriver()` для каждого `order.drivers`.

### Evidence

```text
1: import axios from 'axios'
2: import { Stringify } from '../types'
3: import {
4:   EServices,
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
40:     b_services: data.b_voting && !data.b_services?.includes(EServices.Voting) ?
41:       [...(data.b_services ?? []), EServices.Voting] :
42:       data.b_services,
43:   })
44: 
45:   const params = cloneFormData(formData)
46:   addToFormData(params, {
47:     data: JSON.stringify(Object.fromEntries(
48:       [
49:         'b_start_address',
50:         'b_start_latitude',
51:         'b_start_longitude',
52:         'b_destination_address',
53:         'b_destination_latitude',
54:         'b_destination_longitude',
55:         'b_start_datetime',
56:         'b_custom_comment',
57:         'b_flight_number',
58:         'b_terminal',
59:         'b_passengers_count',
60:         'b_luggage_count',
61:         'b_placard',
62:         'b_car_class',
63:         'b_payment_way',
64:         'b_payment_card',
65:         'b_cars_count',
66:         'b_max_waiting',
67:         'b_options',
68:         'b_contact',
69:         'b_comments',
70:         'b_services',
71:         'b_location_class',
72:         'b_currency',
73:       ].filter(key => key in converted).map(key => [key, converted[key]]),
74:     )),
75:   })
76: 
77:   const response = await axios.post(`${Config.API_URL}/drive`, params)
78:   if (response.data.status === 'error')
79:     throw response.data
80:   const result = response.data.data
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
143:     case EOrderTypes.Ready:
144:       URLAdditionalPath = '/now'
145:       break
146:     case EOrderTypes.History:
147:       URLAdditionalPath = '/archive'
148:       break
149:     default:
150:       return Promise.reject()
151:   }
152:   if (filter.carClasses)
153:     queryParams.push('filter=b_car_classes')
154:   if (filter.locationClasses)
155:     queryParams.push('filter=b_location_classes')
156: 
157:   const { data: response } = await axios.post(
158:     `${Config.API_URL}/drive${URLAdditionalPath}` +
159:     (queryParams.length > 0 ? `?${queryParams.join('&')}` : ''),
160:     formData,
161:   )
162:   if (response.code === '404' && [
163:     'used car not found',
164:     'empty driver location classes',
165:   ].includes(response.message))
166:     return { ...response, data: { detail: 'used_car_not_found' } }
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

```text
1: import moment from 'moment'
2: import {
3:   IBookingCoordinates, IBookingAddresses, IAddressPoint,
4:   IOrder, IDriver, ICar, IUser, ITrip,
5: } from '../types/types'
6: 
7: const dateFormat = 'YYYY-MM-DD HH:mm:ssZ'
8: 
9: export const convertDriver = (driver: any): IDriver => {
10:   return convertTypes<any, IDriver>(
11:     driver,
12:     {
13:       toIntKeys: [
14:         'c_state',
15:       ],
16:       toFloatKeys: [
17:         'c_latitude',
18:         'c_longitude',
19:       ],
20:       toDateKeys: [
21:         'l_datetime',
22:       ],
23:       toBooleanKeys: [
24:         'c_arrive_state',
25:       ],
26:     },
27:   )
28: }
29: 
30: export const reverseConvertDriver = (driver: IDriver): any => {
31:   return convertTypes<any, IDriver>(
32:     driver,
33:     {
34:       toStringKeys: [
35:         'c_latitude',
36:         'c_longitude',
37:         'c_state',
38:       ],
39:       toStringDateKeys: [
40:         'l_datetime',
41:       ],
42:       toIntBooleanKeys: [
43:         'c_arrive_state',
44:       ],
45:     },
46:   )
47: }
48: 
49: export const convertOrder = (order: any): IOrder => {
50:   return convertTypes<any, IOrder>(
51:     order,
52:     {
53:       toFloatKeys: [
54:         'b_start_latitude',
55:         'b_start_longitude',
56:         'b_destination_latitude',
57:         'b_destination_longitude',
58:         'b_passengers_count',
59:         'b_luggage_count',
60:         'b_price_estimate',
61:       ],
62:       toIntKeys: [
63:         'b_cars_count',
64:         'b_distance_estimate',
65:         'b_car_class',
66:         'b_state',
67:       ],
68:       toDateKeys: [
69:         'b_start_datetime',
70:         'b_created',
```

`convertDriver()` явно преобразует:

```text
c_latitude → float
c_longitude → float
l_datetime → date
```

Таким образом для этого frontend snapshot подтверждено:

```text
Core Backend order response
    ↓
drivers[]
    ↓
IDriver
    ↓
c_latitude / c_longitude / l_datetime
```

Это закрывает прежний `SOURCE_GAP` на уровне **Backend → Frontend data exposure**.

## 7. Но Passenger Map не использует координаты Driver

`Passenger` получает `selectedOrder` и `selectedOrderDriver`; выбор `selectedOrderDriver` использует `c_state`.

### Evidence

```text
45: 
46: function Passenger({
47:   activeOrders,
48:   selectedOrder: selectedOrderID,
49:   user,
50:   setVoteModal,
51:   setDriverModal,
52:   setMessageModal,
53:   setOnTheWayModal,
54:   setRatingModal,
55:   setCandidatesModal,
56:   watchActiveOrders,
57:   setFrom,
58:   setTo,
59:   setSelectedOrder,
60: }: IProps) {
61: 
62:   const mapCenter = useRef<[lat: number, lng: number]>(null)
63:   const setMapCenter = useCallback((value: [number, number]) => {
64:     mapCenter.current = value
65:   }, [])
66: 
67:   const formContainerRef = useRef<HTMLDivElement>(null)
68:   const draggableRef = useRef<HTMLDivElement>(null)
69:   const minimizedPartRef = useRef<HTMLElement>(null)
70:   const formSlidersRef = useRef<HTMLElement[]>([])
71:   const { isExpanded, setIsExpanded } = useSwipe(
72:     formContainerRef, draggableRef,
73:     minimizedPartRef, formSlidersRef,
74:   )
75: 
76:   const setFromAsMapCenter = useCallback(() => {
77:     if (isExpanded)
78:       setIsExpanded(false)
79:     else if (mapCenter.current) {
80:       const [latitude, longitude] = mapCenter.current
81:       setFrom({ latitude, longitude })
82:     }
83:   }, [isExpanded])
84:   const setToAsMapCenter = useCallback(() => {
85:     if (isExpanded)
86:       setIsExpanded(false)
87:     else if (mapCenter.current) {
88:       const [latitude, longitude] = mapCenter.current
89:       setTo({ latitude, longitude })
90:     }
91:   }, [isExpanded])
92: 
93:   const selectedOrder = useMemo(() =>
94:     activeOrders?.find((item) => item.b_id === selectedOrderID) ?? null
95:   , [activeOrders, selectedOrderID])
96:   const selectedOrderDriver = useMemo(() =>
97:     selectedOrder?.drivers &&
98:     selectedOrder?.drivers.find(
99:       (item) => item.c_state > EBookingDriverState.Canceled,
100:     )
101:   , [selectedOrder])
102: 
103:   useEffect(watchActiveOrders, [])
104: 
105:   const openCurrentModal = () => {
106:     if (!selectedOrder) {
107:       setVoteModal(false)
108:       setDriverModal(false)
109:       setOnTheWayModal(false)
110:       setCandidatesModal(false)
111:     }
112: 
113:     else if (
114:       candidateMode(selectedOrder) && !selectedOrderDriver &&
115:       (selectedOrder.drivers?.length ?? 0) > 0
116:     )
117:       setCandidatesModal(true)
118: 
119:     else if (selectedOrder.b_voting && !selectedOrderDriver)
120:       setVoteModal(true)
121: 
122:     else
123:       onDriverStateChange()
124:   }
125: 
```

Поиск по всему frontend source показывает, что реальные occurrences:

```text
c_latitude
c_longitude
```

есть только в:

```text
types.ts
convert.ts
```

Дополнительных UI consumers не найдено.

То есть:

```text
Driver coordinates
    → received
    → typed
    → converted
    → NOT CONSUMED by Passenger Map
```

## 8. Passenger Map использует собственную позицию пользователя

`src/components/Map/index.tsx` получает:

```text
navigator.geolocation
    ↓
userCoordinates
```

и отображает:

```text
CircleMarker
Circle accuracy
```

### Evidence

```text
55: }: IProps) {
56:   return (
57:     <div
58:       className={cn('map-container', containerClassName, { 'map-container--active': isOpen, 'map-container--modal': isModal })}
59:       key={SITE_CONSTANTS.MAP_MODE}
60:     >
61:       <MapContainer
62:         center={defaultCenter || SITE_CONSTANTS.DEFAULT_POSITION}
63:         zoom={defaultZoom}
64:         className='map'
65:         attributionControl={false}
66:       >
67:         <MapContent
68:           {...{ isOpen, defaultCenter, isModal, containerClassName }}
69:           {...props}
70:         />
71:       </MapContainer>
72:     </div>
73:   )
74: }
75: 
76: function MapContent({
77:   isOpen = true,
78:   type,
79:   defaultCenter,
80:   clientFrom,
81:   clientTo,
82:   detailedOrderStart,
83:   detailedOrderDestination,
84:   takePassengerFrom,
85:   takePassengerTo,
86:   disableButtons,
87:   isModal,
88:   onClose,
89:   containerClassName,
90:   setCenter = () => {},
91: }: IProps) {
92: 
93:   const map = useMap()
94: 
95:   const [staticMarkers, setStaticMarkers] = useState<IStaticMarker[]>([])
96:   const [userCoordinates, setUserCoordinates] =
97:     useState<IAddressPoint | null>(null)
98:   const [userCoordinatesAccuracy, setUserCoordinatesAccuracy] =
99:     useState<number | null>(null)
100:   const [routeInfo, setRouteInfo] = useState<IRouteInfo | null>(null)
101:   const [showRouteInfo, setShowRouteInfo] = useState(false)
102: 
103:   let from: IAddressPoint | null = null,
104:     to: IAddressPoint | null = null
105:   switch (type) {
106:     case EMapModalTypes.Client:
107:       from = clientFrom
108:       to = clientTo
109:       break
110:     case EMapModalTypes.OrderDetails:
111:       from = detailedOrderStart
112:       to = detailedOrderDestination
113:       break
114:     case EMapModalTypes.TakePassenger:
115:       from = takePassengerFrom || null
116:       to = takePassengerTo || null
117:       break
118:     default:
119:       console.error('Wrong map type:', type)
120:       break
121:   }
122: 
123:   useEffect(() => {
124:     if (isOpen) {
125:       API.getWashTrips()
126:         .then(items => items.filter(item =>
127:           // @ts-ignore
128:           item.t_start_latitude && item.t_start_latitude === item.t_destination_latitude &&
129:           // @ts-ignore
130:           item.t_start_datetime?.format && item.t_complete_datetime?.format &&
131:           // @ts-ignore
132:           item.t_complete_datetime.isAfter(Date.now()),
133:         ))
134:         .then(items => {
135:           // @ts-ignore
136:           const markers = items.map(item => ({
137:             // @ts-ignore
138:             latitude: item.t_start_latitude,
139:             // @ts-ignore
140:             longitude: item.t_start_longitude,
141:             // @ts-ignore
142:             popup: `from ${item.t_start_datetime.format('HH:mm MM-DD')} to ${item.t_complete_datetime.format('HH:mm MM-DD')}`,
143:             // @ts-ignore
144:             tooltip: `until ${item.t_complete_datetime.format('HH:mm MM-DD')}`,
145:           }))
```

Это позиция текущего пользователя.

Нет data-flow:

```text
selectedOrderDriver.c_latitude
    ↓
Map
```

## 9. Driver Map также использует browser geolocation

`src/pages/Driver/Map.tsx` использует:

```text
navigator.geolocation.getCurrentPosition()
```

и строит:

```text
currentPosition
    ↓
Marker
    ↓
Polyline
```

### Evidence

```text
70:       >
71:         <DriverOrderMapModeContent
72:           {...props}
73:           locate={!position}
74:           {...{ setPosition, setZoom }}
75:         />
76:       </MapContainer>
77:     </PageSection>
78:   )
79: }
80: 
81: interface IContentProps extends IProps {
82:   locate: boolean,
83:   setZoom: (zoom: number) => void
84:   setPosition: (position: L.LatLngExpression) => void
85: }
86: 
87: function DriverOrderMapModeContent({
88:   user,
89:   activeOrders,
90:   readyOrders,
91:   locate,
92:   setPosition,
93:   setZoom,
94:   getOrder,
95:   setRatingModal,
96:   setMessageModal,
97:   setOrderCardModal,
98: }: IContentProps) {
99: 
100:   const navigate = useNavigate()
101:   const map = useMap()
102: 
103:   const [lastPositions, setLastPositions] = useState<[number, number][]>([])
104:   // Заместо useState используем useRef чтобы не пересоздавать иконку каждый раз
105:   const arrowIconRef = useRef(
106:     new L.DivIcon({
107:       className: 'driver-arrow-divicon',
108:       iconAnchor: [20, 40],
109:       popupAnchor: [0, -35],
110:       iconSize: [40, 40],
111:       // TODO: Убрать id, сделать стили через класс
112:       html: `
113:         <img
114:           id="driver-arrow"
115:           src="${images.mapArrow}"
116:           style="
117:             transition: transform 0.15s linear;
118:             display: block;
119:             width: 100%;
120:             height: auto;
121:           "
122:         />
123:       `,
124:     }),
125:   )
126: 
127:   useEffect(() => {
128:     if (map) {
129:       map.once('locationfound', (e: L.LocationEvent) => {
130:         setLastPositions([[e.latlng.lat, e.latlng.lng]])
131:         if (locate)
132:           map.setView(e.latlng)
133:       })
134:       map.once('locationerror', (e: L.ErrorEvent) => console.error(e.message))
135:       map.locate({
136:         timeout: Infinity,
137:         enableHighAccuracy: true,
138:       })
139: 
140:       map.on(
141:         'click',
142:         (e: L.LeafletMouseEvent) => {
143:           if (!(e.originalEvent?.target as HTMLDivElement)?.classList?.contains('map')) return
144: 
145:           if (user && window.confirm(`${t(TRANSLATION.CONFIRM_LOCATION)}?`)) {
146:             API.notifyPosition({ latitude: e.latlng.lat, longitude: e.latlng.lng })
147:           }
148:         },
149:       )
150:       map.on(
151:         'zoomend', () => {
152:           setZoom(map.getZoom())
153:         },
154:       )
155:       map.on(
156:         'moveend', () => {
157:           setPosition(map.getCenter())
158:         },
159:       )
160:     }
161:   }, [map])
162: 
163:   useInterval(() => {
164:     navigator.geolocation.getCurrentPosition(
165:       ({ coords }) => {
166:         setLastPositions(prev => {
167:           if (prev.length) {
168:             let newPositions = [
169:               ...prev.reverse().slice(0, 2).reverse(),
170:               [coords.latitude, coords.longitude],
171:             ]
172:             const p1 = newPositions[newPositions.length - 2]
173:             const p2 = newPositions[newPositions.length - 1]
174:             const angle = getAngle(
175:               {
```

Это собственная позиция Driver в браузере.

Она не берётся из `driver.c_latitude / driver.c_longitude`.

## 10. Текущая AS-IS модель

```text
Browser Geolocation
        │
        ↓
Frontend Geolocation State
        │
        ↓
POST /location
        │
        ↓
Core Backend
        │
        ↓
Driver location
        │
        ↓
Order response / drivers[]
        │
        ↓
Frontend IDriver
   ├── u_id
   ├── c_latitude
   ├── c_longitude
   └── l_datetime
```

Одновременно:

```text
Passenger Map
    ↓
navigator.geolocation
    ↓
current user's position
```

и:

```text
Driver Map
    ↓
navigator.geolocation
    ↓
driver's own browser position
```

Но:

```text
IDriver.c_latitude/c_longitude
    ↓
Passenger Map
```

в текущем source не найдено.

## 11. Что теперь CONFIRMED

```text
User
  → HAS_CURRENT_POSITION
  → Browser Geolocation

Frontend Geolocation
  → WRITES
  → Core Backend /location

Order
  → HAS_DRIVER
  → IDriver

IDriver
  → HAS_POSITION
  → c_latitude/c_longitude

IDriver
  → HAS_POSITION_TIMESTAMP
  → l_datetime

Order response
  → EXPOSES_DRIVER_POSITION
  → Taxi Frontend
```

Последний Claim относится именно к `taxi-master.zip` snapshot.

## 12. Что остаётся UNKNOWN

```text
Driver Position
    → RENDERED_BY
    → Passenger Map
```

Статус:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Это уже **не SOURCE_GAP**:

- frontend source существует;
- нужные поля существуют;
- conversion существует;
- source corpus исследован;
- consumer в Passenger Map не найден.

## 13. Важное методологическое различие

RP-29 практически подтверждает различие:

```text
UNKNOWN / SOURCE_GAP
```

и:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

В данном случае:

```text
Frontend source exists
        ↓
field exists
        ↓
field reaches frontend model
        ↓
consumer not found
        ↓
BEHAVIOR_UNRESOLVED
```

## 14. Новая Research Question

Теперь вопрос:

> Почему frontend получает `c_latitude/c_longitude` Driver в `IDriver`, но Passenger Map не использует эти поля?

Expected Evidence:

```text
c_latitude/c_longitude
    ↓
consumer
    ↓
Passenger UI
    ↓
Map marker
```

Possible results:

```text
CONFIRMED
REJECTED
SOURCE_GAP
BEHAVIOR_UNRESOLVED
```

## 15. Многофронтовая модель

Теперь хорошо видна граница:

```text
Core Backend
    │
    ├── exposes Driver Position
    │
    ├── Frontend Snapshot F1
    │       └── receives but does not consume in Map
    │
    ├── Frontend Snapshot F2
    │       └── potentially different behavior
    │
    └── Frontend Snapshot F3
            └── potentially different behavior
```

Поэтому Claim:

```text
Frontend uses Driver Position
```

должен иметь:

```text
backend_snapshot
+
frontend_snapshot
+
consumer evidence
```

## 16. MCR

`MCR = NO CHANGE`.

RP-29 подтверждает уже существующую provenance/snapshot boundary и не требует изменения фундаментальной структуры Semantic Graph.

## 17. Следующий шаг

Не возвращаться к `/location` backend.

Следующий pass должен быть узким frontend consumer audit:

```text
c_latitude
c_longitude
    ↓
ALL consumers
    ↓
Passenger UI
    ↓
Map
```

Если consumer действительно отсутствует, можно будет сформулировать сильный AS-IS вывод:

> Core Backend уже предоставляет Taxi Frontend координаты водителя через `Order.drivers[].c_latitude/c_longitude`, однако текущий Passenger Map эти координаты не визуализирует.

Это уже будет не гипотеза и не `SOURCE_GAP`, а результат прямого анализа конкретного frontend snapshot.
