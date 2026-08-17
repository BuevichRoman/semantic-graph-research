# Backend Semantic Graph — Research Pass 34
# Taxi Frontend Login Endpoint Trace v0.1

**Статус:** EVIDENCE-GROUNDED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-33 Taxi Frontend Authentication Concrete Flow v0.1  
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

## 1. Research Question

> Есть ли в конкретном Taxi Frontend snapshot один явно определяемый login/auth endpoint, через который можно восстановить получение credentials и последующее использование их в Core Backend API?

Критерий остаётся строгим:

```text
Frontend login call
      ↓
concrete backend endpoint
      ↓
response credential
      ↓
frontend persistence/state
      ↓
ordinary protected request
```

## 2. API-layer auth candidates

### `src/API/auth.ts:56`
```text
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

### `src/API/auth.ts:66`
```text
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
```

### `src/API/auth.ts:69`
```text
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
```

### `src/API/auth.ts:117`
```text
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
```

### `src/API/auth.ts:122`
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

### `src/API/auth.ts:129`
```text
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
```

### `src/API/auth.ts:144`
```text
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
```

### `src/API/auth.ts:158`
```text
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
```

### `src/API/auth.ts:159`
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
```

### `src/API/auth.ts:161`
```text
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
```

### `src/API/auth.ts:188`
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
```

### `src/API/index.ts:33`
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
```

### `src/API/index.ts:37`
```text
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
```

## 3. Frontend login/auth call sites

### `src/components/modals/LoginModal/Login.tsx:188`
```text
180:     }
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

### `src/components/modals/LoginModal/Login.tsx:201`
```text
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
204: 
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
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
```

### `src/components/modals/LoginModal/Login.tsx:304`
```text
296:         //   //   u_phone: '',
297:         //   //   u_email: 'moj14frffefff@gmail.com',          // TODO: заменить на data?.email
298:         //   //   type: ERegistrationType.Email,
299:         //   //   u_role: EUserRoles.Client,
300:         //   //   ref_code: '',
301:         //   //   u_details: {},
302:         //   //   st: '1',
303:         //   // }
304:         //   //googleLogin(obj)
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
319:         fixedSize={false}
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:96`
```text
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
100: 
101:     API.checkRefCode(formData.code).then(isFreeCode => {
102:       if (isFreeCode) {
103:         setError('code', { type: 'custom', message: t(TRANSLATION.REF_CODE_NOT_FOUND) })
104:         return
105:       }
106:       data.ref_code = formData.code
107: 
108:       if(whatsappSignUpData?.u_phone) {
109:         whatsappSignUp({
110:           type: ERegistrationType.Whatsapp,
111:           login: whatsappSignUpData.u_phone,
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:117`
```text
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
120:     })
121:   }
122: 
123:   return (
124:     <Overlay
125:       isOpen={payload.isOpen}
126:     >
127:       <div
128:         className="modal refcode-modal"
129:       >
130:         <form onSubmit={handleSubmit(onSubmit)}>
131:           <fieldset>
132:             <div className="code-block">
```

### `src/components/modals/LoginModal/WACodeModal.tsx:78`
```text
70:     criteriaMode: 'all',
71:     mode: 'all',
72:     resolver: yupResolver(schema),
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
87:       <div
88:         className="modal whatsapp-modal"
89:       >
90:         <form onSubmit={handleSubmit(onSubmit)}>
91:           <fieldset>
92:             <div className="code-block">
93:               <p>{t(TRANSLATION.CODE_INFO)}</p>
```

## 4. Что считается login endpoint

Не достаточно обнаружить:

```text
token
u_hash
login
```

в произвольном месте.

Нужна конкретная функция/API adapter, которая принимает authentication input и вызывает backend endpoint.

Только этот endpoint может стать anchor для следующей backend/frontend трассы.

## 5. Current status

На этом проходе зафиксированы конкретные auth-related API contexts и call sites.

Если среди них нет однозначного:

```text
login(credentials)
    → API request
```

то login endpoint нельзя реконструировать по соседним token usages.

В этом случае статус:

```text
UNKNOWN / SOURCE_GAP
```

а не `CONFIRMED`.

## 6. Следующий шаг

После идентификации конкретного endpoint нужно открыть соответствующий Core Backend handler и сопоставить:

```text
frontend request fields
        ↕
backend input
        ↓
authentication logic
        ↓
response fields
        ↓
frontend state
```

Особенно проверить:

```text
token
u_hash
id_user
id_role
```

но только если эти поля реально входят в один и тот же value-flow.

## 7. Graph boundary

Пока сохраняется:

```text
Taxi Frontend Snapshot
    → HAS_AUTH_IMPLEMENTATION

Core Backend
    → AUTHENTICATES_API_REQUEST
```

Полная relation:

```text
Taxi Frontend
    → AUTHENTICATES_WITH
    → Core Backend
```

не закрывается до подтверждения endpoint + credential flow.
