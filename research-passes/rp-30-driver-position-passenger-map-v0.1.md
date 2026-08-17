# Backend Semantic Graph — Research Pass 30
# Driver Position → Passenger Map Consumer v0.1

**Статус:** CONFIRMED NEGATIVE / AS-IS  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-29 Taxi Frontend Location / Assigned Driver Trace v0.1  
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

---

## 1. Research Question

> Использует ли текущий Passenger Map координаты назначенного Driver (`IDriver.c_latitude/c_longitude`) для отображения его позиции?

RP-29 установил:

```text
Core Backend
    ↓
Order.drivers[]
    ↓
IDriver
    ├── u_id
    ├── c_latitude
    ├── c_longitude
    └── l_datetime
```

Теперь проверяется именно последний consumer path:

```text
IDriver.c_latitude/c_longitude
        ↓
Passenger state
        ↓
Map props/state
        ↓
Marker
```

---

## 2. Passenger получает assigned Driver

В `src/pages/Passenger/index.tsx` действительно существует:

```text
selectedOrder
    ↓
selectedOrder.drivers
    ↓
selectedOrderDriver
```

Driver выбирается по `c_state`:

```text
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
```

Это подтверждает:

```text
Passenger UI
    → READS
    → Order.drivers[]
```

Но `selectedOrderDriver` используется только для проверки состояния заказа и открытия соответствующих modal/scenario branches.

---

## 3. Проверка использования координат Driver

Весь frontend source был проверен на:

```text
.c_latitude
.c_longitude
```

Результат:

```text
src/types/types.ts
src/tools/convert.ts
```

Других production consumers нет.

Следовательно, после conversion:

```text
raw driver response
    ↓
IDriver.c_latitude/c_longitude
```

данные не проходят дальше в UI rendering path.

---

## 4. Passenger Map получает другой набор данных

`Passenger` рендерит:

```text
<Map
    containerClassName="passenger__form-map-container"
    setCenter={setMapCenter}
 />
```

Map не получает:

```text
selectedOrder
selectedOrderDriver
driver coordinates
```

### Evidence

```text
220:       }
221:   }, [activeOrders])
222: 
223:   return (
224:     <Layout>
225:       <PageSection className="passenger" scrollable={false}>
226: 
227:         {useMemo(() =>
228:           <MiniOrders
229:             className="passenger__mini-orders"
230:             handleOrderClick={handleOrderClick}
231:           />
232:         , [handleOrderClick])}
233: 
234:         {useMemo(() =>
235:           <Map
236:             containerClassName="passenger__form-map-container"
237:             setCenter={setMapCenter}
238:           />
239:         , [setMapCenter])}
240: 
241:         <div className="passenger__form-placeholder" />
242: 
243:         <div
244:           ref={formContainerRef}
245:           className="passenger__form-container"
```

Следовательно, уже на границе:

```text
Passenger
    ↓
Map
```

assigned Driver Position отсутствует в props.

---

## 5. Map component имеет собственный state

`src/components/Map/index.tsx` хранит:

```text
userCoordinates
userCoordinatesAccuracy
routeInfo
staticMarkers
```

### Evidence

```text
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
```

Для position map component использует:

```text
map.locate()
navigator.geolocation.getCurrentPosition()
```

### Evidence

```text
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
146:           setStaticMarkers(markers)
147:         })
148:     }
149:   }, [isOpen])
150: 
151:   useEffect(() => {
152:     if (!map) return
153: 
154:     map.once('locationfound', (e: L.LocationEvent) => {
155:       setUserCoordinates({
156:         latitude: e.latlng.lat,
157:         longitude: e.latlng.lng,
158:       })
159:       setUserCoordinatesAccuracy(e.accuracy)
160:       if (!defaultCenter)
161:         map.setView(e.latlng)
162:     })
163:     map.once('locationerror', (e: L.ErrorEvent) => console.error(e.message))
164:     map.locate({
165:       timeout: Infinity,
166:       enableHighAccuracy: true,
167:     })
168:   }, [map])
169: 
170:   useInterval(() => {
171:     navigator.geolocation.getCurrentPosition(
172:       ({ coords }) => {
173:         setUserCoordinates({
174:           latitude: coords.latitude,
175:           longitude: coords.longitude,
```

То есть это:

```text
Current Browser User
    ↓
userCoordinates
    ↓
Map
```

а не:

```text
Assigned Driver
    ↓
c_latitude/c_longitude
    ↓
Map
```

---

## 6. Какие markers реально рисует Passenger Map

В `MapContent` production rendering path содержит:

```text
userCoordinates
    → CircleMarker

routeInfo.points
    → Polyline

from
    → Marker

to
    → Marker
```

### Evidence

```text
245:             {duration}
246:           </div>
247:         )
248:       }
249:       {
250:         routeInfo && (
251:           <Polyline positions={routeInfo.points} />
252:         )
253:       }
254:       {
255:         !!userCoordinates?.latitude &&
256:         !!userCoordinates?.longitude &&
257:         <CircleMarker
258:           radius={0}
259:           weight={10}
260:           center={[userCoordinates.latitude, userCoordinates.longitude]}
261:         />
262:       }
263:       {
264:         !!userCoordinates?.latitude &&
265:         !!userCoordinates?.longitude &&
266:         !!userCoordinatesAccuracy &&
267:         <Circle
268:           radius={userCoordinatesAccuracy}
269:           center={[userCoordinates.latitude, userCoordinates.longitude]}
270:         />
271:       }
272:       {staticMarkers.map(marker => (
273:         <Marker
274:           position={[marker.latitude, marker.longitude]}
275:           icon={new L.Icon({
276:             iconUrl: images.activeMarker,
277:             iconSize: [24, 34],
278:             iconAnchor: [12, 34],
279:             popupAnchor: [0, -35],
280:           })}
281:         >
282:           {!!marker.tooltip &&
283:             <Tooltip direction="top" offset={[0, -40]} opacity={1} permanent>{marker.tooltip}</Tooltip>
284:           }
285:           {!!marker.popup && <Popup>{marker.popup}</Popup>}
286:         </Marker>
287:       ))}
288:       {!!from?.latitude && !!from?.longitude &&
289:         <Marker
290:           position={[from.latitude, from.longitude]}
291:           icon={new L.Icon({
292:             iconUrl: images.markerFrom,
293:             iconSize: [35, 41],
294:             iconAnchor: [18, 41],
295:             popupAnchor: [0, -35],
296:           })}
297:         >
298:           <Popup>{t(TRANSLATION.FROM)}{!!from.address && `: ${from.shortAddress || from.address}`}</Popup>
299:         </Marker>
300:       }
301:       {!!to?.latitude && !!to?.longitude &&
302:         <Marker
303:           position={[to.latitude, to.longitude]}
304:           icon={new L.Icon({
305:             iconUrl: images.markerTo,
306:             iconSize: [36, 41],
307:             iconAnchor: [18, 41],
308:             popupAnchor: [0, -35],
309:           })}
310:         >
311:           <Popup>{t(TRANSLATION.TO)}{!!to.address && `: ${to.shortAddress || to.address}`}</Popup>
312:         </Marker>
313:       }
314:       <img
315:         src="https://unpkg.com/leaflet@1.6.0/dist/images/marker-icon-2x.png"
```

Ни один из этих rendering paths не получает:

```text
IDriver.c_latitude
IDriver.c_longitude
```

---

## 7. Важное различие: Driver Map

`src/pages/Driver/Map.tsx` действительно имеет Marker и Polyline, но их source:

```text
navigator.geolocation
    ↓
lastPositions
    ↓
currentPosition
    ↓
Marker / Polyline
```

### Evidence

```text
65:       <MapContainer
66:         center={position ?? SITE_CONSTANTS.DEFAULT_POSITION}
67:         zoom={zoom}
68:         className='map'
69:         attributionControl={false}
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
```

и:

```text
185:             if (img)
186:               img.style.transform = `rotate(${angle}deg)`
187:             return newPositions as typeof prev
188:           }
189:           return [[coords.latitude, coords.longitude]] as typeof prev
190:         })
191:       },
192:       error => console.error(error),
193:       { enableHighAccuracy: true },
194:     )
195:   }, 2000)
196: 
197:   const performingOrder = activeOrders
198:     ?.find(item => ([
199:       EBookingDriverState.Performer, EBookingDriverState.Arrived,
200:     ] as any[]).includes(
201:       item.drivers?.find(item => item.u_id === user?.u_id)?.c_state),
202:     )
203: 
204:   const currentOrder = activeOrders
205:     ?.find(item =>
206:       item.drivers?.find(item => item.u_id === user?.u_id)?.c_state === EBookingDriverState.Started,
207:     )
208: 
209:   const onCompleteOrderClick = () => {
210:     if (!currentOrder) return
211: 
212:     API.setOrderState(currentOrder.b_id, EBookingDriverState.Finished)
213:       .then(() => {
214:         getOrder(currentOrder.b_id)
215:         navigate(`/driver-order?tab=${EDriverTabs.Lite}`)
216:         setRatingModal({ isOpen: true })
217:       })
218:       .catch(error => {
219:         console.error(error)
220:         setMessageModal({ isOpen: true, status: EStatuses.Fail, message: t(TRANSLATION.ERROR) })
221:       })
222:   }
223: 
224:   // Мемоизируем текущую позицию маркера (чтобы React не пересоздавал <Marker> из-за новой ссылки на массив)
225:   const currentPosition = useMemo(() => {
226:     if (!lastPositions || !lastPositions.length) return null
227:     const last = lastPositions[lastPositions.length - 1]
228:     return [last[0], last[1]] as L.LatLngExpression
229:   }, [lastPositions])
230: 
231: 
232:   return (
233:     <>
234:       <TileLayer
235:         attribution={getAttribution()}
236:         url={getTileServerUrl()}
237:       />
238:       {
239:         // Заменяем lastPositions.map() на одиночный <Marker> с мемоизированной позицией и arrowIconRef.current
240:         currentPosition && (
241:           <Marker
242:             position={currentPosition}
243:             icon={arrowIconRef.current}
244:             key="driver-arrow"
245:           />
246:         )
247:       }
248:       {
249:         !!lastPositions.length &&
250:         <Polyline positions={lastPositions} />
251:       }
252:       {
253:         [
254:           ...(readyOrders || []),
255:           ...(performingOrder ? [performingOrder] : []),
```

Это собственная позиция Driver Browser.

Поэтому этот компонент не закрывает:

```text
Passenger observes Driver
```

---

## 8. Итоговая AS-IS data-flow

Теперь цепочка разделяется точно:

```text
DRIVER SELF-LOCATION

Driver Browser
    ↓
navigator.geolocation
    ↓
Frontend Geolocation
    ↓
POST /location
    ↓
Core Backend
    ↓
Order.drivers[]
    ↓
IDriver.c_latitude/c_longitude
```

Но дальше:

```text
IDriver.c_latitude/c_longitude
    X
Passenger Map
```

нет consumer.

Параллельно существует:

```text
PASSENGER SELF-LOCATION

Passenger Browser
    ↓
navigator.geolocation
    ↓
Map.userCoordinates
    ↓
CircleMarker
```

И:

```text
DRIVER SELF-MAP

Driver Browser
    ↓
navigator.geolocation
    ↓
Driver Map.lastPositions
    ↓
Marker / Polyline
```

---

## 9. Статус relation

Для текущего Taxi frontend snapshot:

```text
Order
  → HAS_DRIVER
  → IDriver                         CONFIRMED

IDriver
  → HAS_LOCATION_FIELDS
  → c_latitude/c_longitude          CONFIRMED

Passenger
  → READS
  → selectedOrderDriver             CONFIRMED

Passenger Map
  → CONSUMES
  → Driver Position                 REJECTED
```

Последний статус уже не `SOURCE_GAP` и не обычный `UNKNOWN`.

Причина:

1. frontend source присутствует;
2. `IDriver.c_latitude/c_longitude` определены;
3. Passenger получает `selectedOrderDriver`;
4. весь production source проверен на coordinate consumers;
5. `Map` не получает Driver coordinates;
6. rendering path Map использует только browser position, order addresses и route points.

Таким образом, для **данного Passenger Map implementation** связь:

```text
Assigned Driver Position
    → RENDERED_ON
    → Passenger Map
```

не существует в AS-IS.

---

## 10. Что это означает для продуктовой задачи

Это не означает:

> Backend не умеет показывать позицию Driver.

Наоборот:

```text
Backend
    → already exposes Driver Position
```

и frontend её получает в `Order.drivers[]`.

Не реализовано именно:

```text
Passenger Map
    → Driver Position marker
```

Следовательно, потенциальная frontend-доработка не требует сначала изобретать новую Backend capability — по текущему Evidence необходимый data already exists в order response.

Это уже отдельный TO-BE вопрос и не должен попадать в AS-IS graph как существующая связь.

---

## 11. Методологический результат

RP-30 закрывает тот самый Research Loop, который первоначально был `UNKNOWN`:

```text
UNKNOWN
   ↓
Research Question
   ↓
Expected Evidence
   ↓
Frontend source
   ↓
field tracing
   ↓
consumer audit
   ↓
REJECTED
```

Это более сильный результат, чем:

```text
SOURCE_GAP
```

потому что теперь код действительно проверен.

Также подтверждается полезность различения:

```text
SOURCE_GAP
≠
BEHAVIOR_UNRESOLVED
≠
REJECTED
```

В данном случае конечный статус — `REJECTED` для конкретного AS-IS relation.

---

## 12. Graph update

AS-IS graph:

```text
User
  ──HAS_CURRENT_POSITION──> Position

Order
  ──HAS_DRIVER──> Driver/User

Driver
  ──POSITION_FIELDS──> Position

Order Response
  ──EXPOSES──> Driver Position

Taxi Frontend
  ──RECEIVES──> Driver Position

Passenger Map
  ──RENDERS──> Passenger Position
  ──RENDERS──> Route
  ──RENDERS──> Order From/To
```

Не добавляется:

```text
Passenger Map
  ──RENDERS──> Assigned Driver Position
```

Статус relation:

```text
REJECTED / ABSENT IN AS-IS
```

---

## 13. Gap Report

Закрыто:

```text
G-30-01  Driver Position → Passenger Map consumer
         REJECTED / AS-IS absent

G-30-02  Driver coordinates available to frontend
         CONFIRMED

G-30-03  Passenger Map current position source
         CONFIRMED
```

Остаётся:

```text
G-30-04  exact backend provenance of c_latitude/c_longitude
         → already covered by backend research; can be linked by evidence IDs

G-30-05  TO-BE design for displaying assigned Driver
         → separate product/frontend task
```

---

## 14. Следующий шаг

**Не продолжать исследование `/location`.**

Backend → Frontend data path уже достаточно восстановлен.

Следующий рациональный шаг — перейти к следующему domain concept, а не углубляться бесконечно в Position:

```text
Order
  → Driver Assignment
  → Driver Position
  → Passenger Visibility
```

или выбрать следующий самостоятельный блок Frontend, например:

```text
Authentication
```

который особенно хорошо подходит для дальнейшей проверки общей модели:

```text
Core Backend
      +
Frontend Snapshot
      +
Role / User
      +
Capability
```

Именно это даст нам следующий фрагмент общего Semantic Graph без искусственного расширения текущего исследования.
