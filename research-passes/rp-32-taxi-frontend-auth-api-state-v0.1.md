# Backend Semantic Graph — Research Pass 32
# Taxi Frontend Authentication API → State v0.1

**Статус:** EVIDENCE-GROUNDED / PROVISIONAL
**Методология:** Semantic Graph Research Methodology v2.3
**Предшествующий проход:** RP-31 Taxi Frontend Authentication Trace v0.1
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

## 1. Research Question

> Как именно frontend получает authentication credentials и где они становятся частью общего API request context?

Целевая цепочка:

```text
login/auth action
   ↓
Core Backend auth endpoint
   ↓
token / u_hash / user identity
   ↓
frontend state/storage
   ↓
API adapter
   ↓
authenticated request
```

## 2. Candidate auth implementation files

Проанализировано candidate auth/API files: **18**.

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
```

### `src/API/auth.ts:53`
```text
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
```

### `src/API/auth.ts:56`
```text
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
```

### `src/API/auth.ts:60`
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
```

### `src/API/auth.ts:69`
```text
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

### `src/API/auth.ts:72`
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
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
```

### `src/API/auth.ts:79`
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
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
```

### `src/API/auth.ts:86`
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
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
```

### `src/API/auth.ts:93`
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
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
```

### `src/API/auth.ts:101`
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
```

### `src/API/auth.ts:102`
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
```

### `src/API/auth.ts:105`
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
```

### `src/API/auth.ts:106`
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
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
```

### `src/API/auth.ts:107`
```text
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
```

### `src/API/auth.ts:109`
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
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
```

### `src/API/auth.ts:110`
```text
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
```

### `src/API/auth.ts:111`
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
```

### `src/API/auth.ts:117`
```text
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
```

### `src/API/auth.ts:122`
```text
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

### `src/API/auth.ts:129`
```text
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
```

### `src/API/auth.ts:142`
```text
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

### `src/API/auth.ts:157`
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
```

### `src/API/auth.ts:166`
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
```

### `src/API/auth.ts:169`
```text
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

### `src/API/auth.ts:170`
```text
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

### `src/API/auth.ts:171`
```text
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

### `src/API/auth.ts:172`
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
```

### `src/API/auth.ts:174`
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
```

### `src/API/auth.ts:179`
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
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
```

### `src/API/auth.ts:180`
```text
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
```

### `src/API/auth.ts:181`
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
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
```

### `src/API/auth.ts:188`
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
```

### `src/API/auth.ts:189`
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
```

### `src/API/auth.ts:190`
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
```

### `src/API/auth.ts:191`
```text
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

### `src/API/auth.ts:193`
```text
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

### `src/API/auth.ts:194`
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
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
```

### `src/API/auth.ts:195`
```text
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
```

### `src/API/auth.ts:196`
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
```

### `src/API/auth.ts:202`
```text
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

### `src/API/auth.ts:204`
```text
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

### `src/API/auth.ts:207`
```text
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
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/state/user/sagas.ts:3`
```text
1: import { all, takeEvery, put } from 'redux-saga/effects'
2: import { TAction } from '../../types'
3: import { EStatuses, EUserRoles, ITokens } from '../../types/types'
4: import { setCookie, getCookie } from '../../utils/cookies'
5: import { call, putResolve } from '../../tools/sagaUtils'
6: import SITE_CONSTANTS from '../../siteConstants'
7: import * as API from '../../API'
8: import { t, TRANSLATION } from '../../localization'
9: import { ActionTypes as ConfigActionTypes } from '../config/constants'
10: import { clearOrders } from '../orders/actionCreators'
11: import { createUserCar } from '../cars/actionCreators'
12: import {
13:   setLoginModal, setMessageModal, setRefCodeModal,
14: } from '../modals/actionCreators'
15: import { uploadRegisterFiles, uploadFiles } from './helpers'
```

### `src/state/user/sagas.ts:13`
```text
7: import * as API from '../../API'
8: import { t, TRANSLATION } from '../../localization'
9: import { ActionTypes as ConfigActionTypes } from '../config/constants'
10: import { clearOrders } from '../orders/actionCreators'
11: import { createUserCar } from '../cars/actionCreators'
12: import {
13:   setLoginModal, setMessageModal, setRefCodeModal,
14: } from '../modals/actionCreators'
15: import { uploadRegisterFiles, uploadFiles } from './helpers'
16: import { ActionTypes } from './constants'
17: import { setUser } from './actionCreators'
18: 
19: export const saga = function* () {
20:   yield all([
21:     takeEvery(ActionTypes.LOGIN_REQUEST, loginSaga),
22:     takeEvery(ActionTypes.GOOGLE_LOGIN_REQUEST, googleLoginSaga),
23:     takeEvery(ActionTypes.REGISTER_REQUEST, registerSaga),
24:     takeEvery(ActionTypes.LOGOUT_REQUEST, logoutSaga),
25:     takeEvery(ActionTypes.REMIND_PASSWORD_REQUEST, remindPasswordSaga),
```

### `src/state/user/sagas.ts:21`
```text
15: import { uploadRegisterFiles, uploadFiles } from './helpers'
16: import { ActionTypes } from './constants'
17: import { setUser } from './actionCreators'
18: 
19: export const saga = function* () {
20:   yield all([
21:     takeEvery(ActionTypes.LOGIN_REQUEST, loginSaga),
22:     takeEvery(ActionTypes.GOOGLE_LOGIN_REQUEST, googleLoginSaga),
23:     takeEvery(ActionTypes.REGISTER_REQUEST, registerSaga),
24:     takeEvery(ActionTypes.LOGOUT_REQUEST, logoutSaga),
25:     takeEvery(ActionTypes.REMIND_PASSWORD_REQUEST, remindPasswordSaga),
26:     takeEvery(ActionTypes.INIT_USER, initUserSaga),
27:     takeEvery(ActionTypes.WHATSAPP_SIGNUP_REQUEST, whatsappSignUpSaga),
28:     call(handleRedirectSaga),
29:   ])
30: }
31: 
32: function* loginSaga(data: TAction) {
33:   yield put({ type: ActionTypes.LOGIN_START })
```

### `src/state/user/sagas.ts:22`
```text
16: import { ActionTypes } from './constants'
17: import { setUser } from './actionCreators'
18: 
19: export const saga = function* () {
20:   yield all([
21:     takeEvery(ActionTypes.LOGIN_REQUEST, loginSaga),
22:     takeEvery(ActionTypes.GOOGLE_LOGIN_REQUEST, googleLoginSaga),
23:     takeEvery(ActionTypes.REGISTER_REQUEST, registerSaga),
24:     takeEvery(ActionTypes.LOGOUT_REQUEST, logoutSaga),
25:     takeEvery(ActionTypes.REMIND_PASSWORD_REQUEST, remindPasswordSaga),
26:     takeEvery(ActionTypes.INIT_USER, initUserSaga),
27:     takeEvery(ActionTypes.WHATSAPP_SIGNUP_REQUEST, whatsappSignUpSaga),
28:     call(handleRedirectSaga),
29:   ])
30: }
31: 
32: function* loginSaga(data: TAction) {
33:   yield put({ type: ActionTypes.LOGIN_START })
34:   try {
```

### `src/state/user/sagas.ts:24`
```text
18: 
19: export const saga = function* () {
20:   yield all([
21:     takeEvery(ActionTypes.LOGIN_REQUEST, loginSaga),
22:     takeEvery(ActionTypes.GOOGLE_LOGIN_REQUEST, googleLoginSaga),
23:     takeEvery(ActionTypes.REGISTER_REQUEST, registerSaga),
24:     takeEvery(ActionTypes.LOGOUT_REQUEST, logoutSaga),
25:     takeEvery(ActionTypes.REMIND_PASSWORD_REQUEST, remindPasswordSaga),
26:     takeEvery(ActionTypes.INIT_USER, initUserSaga),
27:     takeEvery(ActionTypes.WHATSAPP_SIGNUP_REQUEST, whatsappSignUpSaga),
28:     call(handleRedirectSaga),
29:   ])
30: }
31: 
32: function* loginSaga(data: TAction) {
33:   yield put({ type: ActionTypes.LOGIN_START })
34:   try {
35:     const result = yield* call(API.login, data.payload)
36:     if (!result) throw new Error('wrong login response')
```

### `src/state/user/sagas.ts:32`
```text
26:     takeEvery(ActionTypes.INIT_USER, initUserSaga),
27:     takeEvery(ActionTypes.WHATSAPP_SIGNUP_REQUEST, whatsappSignUpSaga),
28:     call(handleRedirectSaga),
29:   ])
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
44:       throw new Error('wrong password')
```

### `src/state/user/sagas.ts:33`
```text
27:     takeEvery(ActionTypes.WHATSAPP_SIGNUP_REQUEST, whatsappSignUpSaga),
28:     call(handleRedirectSaga),
29:   ])
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
44:       throw new Error('wrong password')
45:     }
```

### `src/state/user/sagas.ts:35`
```text
29:   ])
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
44:       throw new Error('wrong password')
45:     }
46: 
47:     if(result.data !== null && result.data === 'wrong phone') {
```

### `src/state/user/sagas.ts:36`
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
44:       throw new Error('wrong password')
45:     }
46: 
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
```

### `src/state/user/sagas.ts:38`
```text
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
45:     }
46: 
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
```

### `src/state/user/sagas.ts:39`
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
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
51:       return
```

### `src/state/user/sagas.ts:40`
```text
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
47:     if(result.data !== null && result.data === 'wrong phone') {
48:       yield put({ type: ActionTypes.WHATSAPP_SIGNUP_START, payload: data.payload.login })
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
51:       return
52:     }
```

### `src/state/user/sagas.ts:43`
```text
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
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
51:       return
52:     }
53: 
54:     if(result.data !== null && result.data === 'code sent') {
55:       yield put({ type: ActionTypes.LOGIN_WHATSAPP })
```

### `src/state/user/sagas.ts:48`
```text
42:     if(result.data === 'wrong password') {
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
57:     }
58: 
59:     if (result.user === null) throw new Error('wrong login response')
60: 
```

### `src/state/user/sagas.ts:49`
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
57:     }
58: 
59:     if (result.user === null) throw new Error('wrong login response')
60: 
61:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
```

### `src/state/user/sagas.ts:55`
```text
49:       yield put(setLoginModal(false))
50:       yield put(setRefCodeModal({ isOpen: true }))
51:       return
52:     }
53: 
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

### `src/state/user/sagas.ts:59`
```text
53: 
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
68:     }
69: 
70:     yield put({
71:       type: ActionTypes.LOGIN_SUCCESS,
```

### `src/state/user/sagas.ts:61`
```text
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
68:     }
69: 
70:     yield put({
71:       type: ActionTypes.LOGIN_SUCCESS,
72:       payload: result,
73:     })
```

### `src/state/user/sagas.ts:71`
```text
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
75:   } catch (error: any) {
76:     console.error('catch', error)
77:     yield put({
78:       type: ActionTypes.LOGIN_FAIL,
79:       payload: error.message,
80:     })
81:   }
82: }
83: 
```

### `src/state/user/sagas.ts:74`
```text
68:     }
69: 
70:     yield put({
71:       type: ActionTypes.LOGIN_SUCCESS,
72:       payload: result,
73:     })
74:     yield put(setLoginModal(false))
75:   } catch (error: any) {
76:     console.error('catch', error)
77:     yield put({
78:       type: ActionTypes.LOGIN_FAIL,
79:       payload: error.message,
80:     })
81:   }
82: }
83: 
84: function* googleLoginSaga(data: TAction) {
85:   yield put({ type: ActionTypes.GOOGLE_LOGIN_START })
86:   try {
```

### `src/state/user/sagas.ts:78`
```text
72:       payload: result,
73:     })
74:     yield put(setLoginModal(false))
75:   } catch (error: any) {
76:     console.error('catch', error)
77:     yield put({
78:       type: ActionTypes.LOGIN_FAIL,
79:       payload: error.message,
80:     })
81:   }
82: }
83: 
84: function* googleLoginSaga(data: TAction) {
85:   yield put({ type: ActionTypes.GOOGLE_LOGIN_START })
86:   try {
87: 
88:     const result = yield* call(API.googleLogin, data.payload)
89: 
90:     if (!result) throw new Error('Wrong login response')
```

### `src/state/user/sagas.ts:84`
```text
78:       type: ActionTypes.LOGIN_FAIL,
79:       payload: error.message,
80:     })
81:   }
82: }
83: 
84: function* googleLoginSaga(data: TAction) {
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
```

### `src/state/user/sagas.ts:85`
```text
79:       payload: error.message,
80:     })
81:   }
82: }
83: 
84: function* googleLoginSaga(data: TAction) {
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
```

### `src/state/user/sagas.ts:88`
```text
82: }
83: 
84: function* googleLoginSaga(data: TAction) {
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
99:   } catch (error) {
100:     console.error(error)
```

### `src/state/user/sagas.ts:90`
```text
84: function* googleLoginSaga(data: TAction) {
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
99:   } catch (error) {
100:     console.error(error)
101:     yield put({ type: ActionTypes.GOOGLE_LOGIN_FAIL })
102:   }
```

### `src/state/user/sagas.ts:91`
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
99:   } catch (error) {
100:     console.error(error)
101:     yield put({ type: ActionTypes.GOOGLE_LOGIN_FAIL })
102:   }
103: }
```

### `src/state/user/sagas.ts:94`
```text
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
100:     console.error(error)
101:     yield put({ type: ActionTypes.GOOGLE_LOGIN_FAIL })
102:   }
103: }
104: 
105: function* registerSaga(data: TAction) {
106:   yield put({ type: ActionTypes.REGISTER_START })
```

### `src/state/user/sagas.ts:97`
```text
91:     localStorage.setItem('state.user.tokens', JSON.stringify(result.tokens))
92: 
93:     yield put({
94:       type: ActionTypes.GOOGLE_LOGIN_SUCCESS,
95:       payload: result,
96:     })
97:     yield put(setLoginModal(false))
98:     yield put(setRefCodeModal({ isOpen: false }))
99:   } catch (error) {
100:     console.error(error)
101:     yield put({ type: ActionTypes.GOOGLE_LOGIN_FAIL })
102:   }
103: }
104: 
105: function* registerSaga(data: TAction) {
106:   yield put({ type: ActionTypes.REGISTER_START })
107:   try {
108:     const { uploads, u_details, u_car, ...payload } = data.payload
109: 
```

### `src/state/user/sagas.ts:101`
```text
95:       payload: result,
96:     })
97:     yield put(setLoginModal(false))
98:     yield put(setRefCodeModal({ isOpen: false }))
99:   } catch (error) {
100:     console.error(error)
101:     yield put({ type: ActionTypes.GOOGLE_LOGIN_FAIL })
102:   }
103: }
104: 
105: function* registerSaga(data: TAction) {
106:   yield put({ type: ActionTypes.REGISTER_START })
107:   try {
108:     const { uploads, u_details, u_car, ...payload } = data.payload
109: 
110:     const response: any = yield* call(API.register, { ...payload, st: 1 })
111:     if (response.error) {
112:       yield put({ type: ActionTypes.REGISTER_FAIL, payload: response.error })
113:       throw new Error(response.error)
```

### `src/state/user/sagas.ts:116`
```text
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
```

### `src/state/user/sagas.ts:117`
```text
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
```

### `src/state/user/sagas.ts:118`
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
126:         const { passport_photo, driver_license_photo, license_photo, ...details } = u_details
127:         const files = { passport_photo, driver_license_photo, license_photo }
128:         const t = { ...tokens, u_id: response.u_id }
129:         yield* call(uploadFiles, { files, u_details: details, tokens: t })
130:       }
```

### `src/state/user/sagas.ts:120`
```text
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
```

### `src/state/user/sagas.ts:128`
```text
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

### `src/state/user/sagas.ts:129`
```text
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
141:       }))) :
```

### `src/state/user/sagas.ts:134`
```text
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
141:       }))) :
142:       null
143: 
144:     const isCarError = carResponse?.status === 'error'
145:     const messageStatus = isCarError ? EStatuses.Warning : EStatuses.Success
146:     const messageText = isCarError ? TRANSLATION.REGISTER_CAR_FAIL : TRANSLATION.SUCCESS_REGISTER_MESSAGE
```

### `src/state/user/sagas.ts:158`
```text
152:   } catch (error) {
153:     console.error(error)
154:     yield put({ type: ActionTypes.REGISTER_FAIL, payload: error })
155:   }
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
170:   }
```

### `src/state/user/sagas.ts:159`
```text
153:     console.error(error)
154:     yield put({ type: ActionTypes.REGISTER_FAIL, payload: error })
155:   }
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
170:   }
171: }
```

### `src/state/user/sagas.ts:162`
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
170:   }
171: }
172: 
173: function* remindPasswordSaga(data: TAction) {
174:   yield put({ type: ActionTypes.REMIND_PASSWORD_START })
```

### `src/state/user/sagas.ts:164`
```text
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
171: }
172: 
173: function* remindPasswordSaga(data: TAction) {
174:   yield put({ type: ActionTypes.REMIND_PASSWORD_START })
175:   try {
176:     yield* call(API.remindPassword, data.payload)
```

### `src/state/user/sagas.ts:165`
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
173: function* remindPasswordSaga(data: TAction) {
174:   yield put({ type: ActionTypes.REMIND_PASSWORD_START })
175:   try {
176:     yield* call(API.remindPassword, data.payload)
177:     yield put({ type: ActionTypes.REMIND_PASSWORD_SUCCESS })
```

### `src/state/user/sagas.ts:169`
```text
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
173: function* remindPasswordSaga(data: TAction) {
174:   yield put({ type: ActionTypes.REMIND_PASSWORD_START })
175:   try {
176:     yield* call(API.remindPassword, data.payload)
177:     yield put({ type: ActionTypes.REMIND_PASSWORD_SUCCESS })
178:   } catch (error) {
179:     yield put({ type: ActionTypes.REMIND_PASSWORD_FAIL })
180:   }
181: }
```

### `src/state/user/sagas.ts:185`
```text
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
```

### `src/state/user/sagas.ts:186`
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
194:             type: ConfigActionTypes.SET_LANGUAGE,
195:             payload: language,
196:           })
197:         }
198:       }
```

### `src/state/user/sagas.ts:187`
```text
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
```

### `src/state/user/sagas.ts:201`
```text
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
210:     if (user?.u_lang) {
211:       const language = SITE_CONSTANTS.LANGUAGES.find(i => {
212:         return i.id.toString() === user.u_lang?.toString()
213:       })
```

### `src/state/user/sagas.ts:203`
```text
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
210:     if (user?.u_lang) {
211:       const language = SITE_CONSTANTS.LANGUAGES.find(i => {
212:         return i.id.toString() === user.u_lang?.toString()
213:       })
214:       if (language) {
215:         setCookie('user_lang', language.iso)
```

### `src/state/user/sagas.ts:205`
```text
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
210:     if (user?.u_lang) {
211:       const language = SITE_CONSTANTS.LANGUAGES.find(i => {
212:         return i.id.toString() === user.u_lang?.toString()
213:       })
214:       if (language) {
215:         setCookie('user_lang', language.iso)
216:         yield put({
217:           type: ConfigActionTypes.SET_LANGUAGE,
```

### `src/state/user/sagas.ts:237`
```text
231:         }
232:       }
233:     }
234:     yield put(setUser(user))
235:   } catch (error) {
236:     console.error('Error in initUserSaga:', error)
237:     localStorage.removeItem('state.user.tokens')
238:   }
239: }
240: 
241: function* whatsappSignUpSaga(data: TAction) {
242:   try {
243:     const result = yield* call(API.whatsappSignUp, data.payload)
244:     if (!result) throw new Error('Wrong whatsappSignUp response')
245: 
246:     yield put(setRefCodeModal({ isOpen: false }))
247:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_SUCCESS, payload: result })
248:     console.log('запускаем loginSaga')
249:     yield* call(loginSaga, {
```

### `src/state/user/sagas.ts:248`
```text
242:   try {
243:     const result = yield* call(API.whatsappSignUp, data.payload)
244:     if (!result) throw new Error('Wrong whatsappSignUp response')
245: 
246:     yield put(setRefCodeModal({ isOpen: false }))
247:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_SUCCESS, payload: result })
248:     console.log('запускаем loginSaga')
249:     yield* call(loginSaga, {
250:       type: ActionTypes.LOGIN_REQUEST,
251:       payload: {
252:         login: data.payload.login,
253:         type: data.payload.type,
254:         navigate: data.payload.navigate,
255:       },
256:     })
257: 
258:   } catch (error) {
259:     console.error(error)
260:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_FAIL })
```

### `src/state/user/sagas.ts:249`
```text
243:     const result = yield* call(API.whatsappSignUp, data.payload)
244:     if (!result) throw new Error('Wrong whatsappSignUp response')
245: 
246:     yield put(setRefCodeModal({ isOpen: false }))
247:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_SUCCESS, payload: result })
248:     console.log('запускаем loginSaga')
249:     yield* call(loginSaga, {
250:       type: ActionTypes.LOGIN_REQUEST,
251:       payload: {
252:         login: data.payload.login,
253:         type: data.payload.type,
254:         navigate: data.payload.navigate,
255:       },
256:     })
257: 
258:   } catch (error) {
259:     console.error(error)
260:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_FAIL })
261:   }
```

### `src/state/user/sagas.ts:250`
```text
244:     if (!result) throw new Error('Wrong whatsappSignUp response')
245: 
246:     yield put(setRefCodeModal({ isOpen: false }))
247:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_SUCCESS, payload: result })
248:     console.log('запускаем loginSaga')
249:     yield* call(loginSaga, {
250:       type: ActionTypes.LOGIN_REQUEST,
251:       payload: {
252:         login: data.payload.login,
253:         type: data.payload.type,
254:         navigate: data.payload.navigate,
255:       },
256:     })
257: 
258:   } catch (error) {
259:     console.error(error)
260:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_FAIL })
261:   }
262: }
```

### `src/state/user/sagas.ts:252`
```text
246:     yield put(setRefCodeModal({ isOpen: false }))
247:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_SUCCESS, payload: result })
248:     console.log('запускаем loginSaga')
249:     yield* call(loginSaga, {
250:       type: ActionTypes.LOGIN_REQUEST,
251:       payload: {
252:         login: data.payload.login,
253:         type: data.payload.type,
254:         navigate: data.payload.navigate,
255:       },
256:     })
257: 
258:   } catch (error) {
259:     console.error(error)
260:     yield put({ type: ActionTypes.WHATSAPP_SIGNUP_FAIL })
261:   }
262: }
263: 
264: function* handleRedirectSaga() {
```

### `src/state/user/sagas.ts:269`
```text
263: 
264: function* handleRedirectSaga() {
265:   const params = new URLSearchParams(window.location.search)
266:   const authHash = params.get('auth_hash')
267:   if (authHash)
268:     yield put({
269:       type: ActionTypes.GOOGLE_LOGIN_REQUEST,
270:       payload: {
271:         data: null,
272:         auth_hash: decodeURIComponent(authHash),
273:       },
274:     })
275: }
```

### `src/components/modals/LoginModal/Login.tsx:12`
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
20: import { useLocation, useNavigate } from 'react-router-dom'
21: import { modalsActionCreators,  modalsSelectors } from '../../../state/modals'
22: import { Input } from './elements'
23: 
24: 
```

### `src/components/modals/LoginModal/Login.tsx:19`
```text
13: import { emailRegex, phoneRegex } from '../../../tools/utils'
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
28:   tab: userSelectors.tab(state),
29:   message: userSelectors.message(state),
30:   isWAOpen: modalsSelectors.isWACodeModalOpen,
31: })
```

### `src/components/modals/LoginModal/Login.tsx:34`
```text
28:   tab: userSelectors.tab(state),
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
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
44: }
45: 
46: const connector = connect(mapStateToProps, mapDispatchToProps)
```

### `src/components/modals/LoginModal/Login.tsx:35`
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
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
44: }
45: 
46: const connector = connect(mapStateToProps, mapDispatchToProps)
47: 
```

### `src/components/modals/LoginModal/Login.tsx:36`
```text
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
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
44: }
45: 
46: const connector = connect(mapStateToProps, mapDispatchToProps)
47: 
48: interface IFormValues {
```

### `src/components/modals/LoginModal/Login.tsx:37`
```text
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
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
44: }
45: 
46: const connector = connect(mapStateToProps, mapDispatchToProps)
47: 
48: interface IFormValues {
49:     login: string | undefined,
```

### `src/components/modals/LoginModal/Login.tsx:49`
```text
43:   setRefOpen: modalsActionCreators.setRefCodeModal,
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
58: 
59: const LoginForm: React.FC<IProps> = ({
60:   user,
61:   status,
```

### `src/components/modals/LoginModal/Login.tsx:59`
```text
53: 
54: interface IProps extends ConnectedProps<typeof connector> {
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
69:   remindPassword,
70:   setMessage,
71:   setStatus,
```

### `src/components/modals/LoginModal/Login.tsx:63`
```text
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
69:   remindPassword,
70:   setMessage,
71:   setStatus,
72:   setLoginModal,
73:   message,
74: }) => {
75:   const [isPasswordShows, setIsPasswordShows] = useState(false)
```

### `src/components/modals/LoginModal/Login.tsx:67`
```text
61:   status,
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
76:   const [dataToLogin, setDataToLogin] = useState({})
77:   const [isVisible, toggleVisibility] = useVisibility(false)
78:   const [isPasswordVisible, togglePasswordVisibility] = useVisibility(true)
79:   const location = useLocation()
```

### `src/components/modals/LoginModal/Login.tsx:68`
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
76:   const [dataToLogin, setDataToLogin] = useState({})
77:   const [isVisible, toggleVisibility] = useVisibility(false)
78:   const [isPasswordVisible, togglePasswordVisibility] = useVisibility(true)
79:   const location = useLocation()
80:   const navigate = useNavigate()
```

### `src/components/modals/LoginModal/Login.tsx:72`
```text
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
77:   const [isVisible, toggleVisibility] = useVisibility(false)
78:   const [isPasswordVisible, togglePasswordVisibility] = useVisibility(true)
79:   const location = useLocation()
80:   const navigate = useNavigate()
81:   const googleClientId = '973943716904-b33r11ijgi08m5etsg5ndv409shh1tjl.apps.googleusercontent.com'
82: 
83:   const role = !location.pathname.includes('/driver-order') ?
84:     EUserRoles.Client :
```

### `src/components/modals/LoginModal/Login.tsx:76`
```text
70:   setMessage,
71:   setStatus,
72:   setLoginModal,
73:   message,
74: }) => {
75:   const [isPasswordShows, setIsPasswordShows] = useState(false)
76:   const [dataToLogin, setDataToLogin] = useState({})
77:   const [isVisible, toggleVisibility] = useVisibility(false)
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
```

### `src/components/modals/LoginModal/Login.tsx:89`
```text
83:   const role = !location.pathname.includes('/driver-order') ?
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
98:     register: formRegister,
99:     handleSubmit,
100:     formState: { errors, isDirty },
101:     control,
```

### `src/components/modals/LoginModal/Login.tsx:107`
```text
101:     control,
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
116:       setStatus(EStatuses.Default)
117:       setMessage('')
118:     }
119:     if (isOpen && isVisible) {
```

### `src/components/modals/LoginModal/Login.tsx:112`
```text
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
119:     if (isOpen && isVisible) {
120:       toggleVisibility()
121:     }
122:   }, [isOpen])
123: 
124:   useEffect(() => {
```

### `src/components/modals/LoginModal/Login.tsx:166`
```text
160:       (status === EStatuses.Fail || status === EStatuses.Success && user) &&
161:       type !== ERegistrationType.Whatsapp && !isVisible
162:     ) {
163:       console.log('togglin, prev: ', isVisible)
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
178:         toggleVisibility()
```

### `src/components/modals/LoginModal/Login.tsx:169`
```text
163:       console.log('togglin, prev: ', isVisible)
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
178:         toggleVisibility()
179:       }
180:     }
181:   }, [status])
```

### `src/components/modals/LoginModal/Login.tsx:170`
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
178:         toggleVisibility()
179:       }
180:     }
181:   }, [status])
182: 
```

### `src/components/modals/LoginModal/Login.tsx:183`
```text
177:       if (!isVisible) {
178:         toggleVisibility()
179:       }
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
```

### `src/components/modals/LoginModal/Login.tsx:188`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:190`
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
198:           ...data,
199:           login: data.login || '',
200:         }
201:       login({ ...loginData, navigate: navigate })
202:     }
```

### `src/components/modals/LoginModal/Login.tsx:192`
```text
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
204: 
```

### `src/components/modals/LoginModal/Login.tsx:195`
```text
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
204: 
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
207:     return value && decodeURIComponent(value)
```

### `src/components/modals/LoginModal/Login.tsx:199`
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
```

### `src/components/modals/LoginModal/Login.tsx:201`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:212`
```text
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
217:           ...formRegister('login'),
218:           placeholder: type === ERegistrationType.Phone || type === ERegistrationType.Whatsapp ?
219:             t(TRANSLATION.PHONE) :
220:             t(TRANSLATION.EMAIL),
221:         }}
222:         label={t(TRANSLATION.LOGIN)}
223:         error={errors.login?.message}
224:         key={type}
```

### `src/components/modals/LoginModal/Login.tsx:217`
```text
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
221:         }}
222:         label={t(TRANSLATION.LOGIN)}
223:         error={errors.login?.message}
224:         key={type}
225:       />
226:       {isPasswordVisible &&
227:             <Input
228:               inputProps={{
229:                 ...formRegister('password'),
```

### `src/components/modals/LoginModal/Login.tsx:222`
```text
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
226:       {isPasswordVisible &&
227:             <Input
228:               inputProps={{
229:                 ...formRegister('password'),
230:                 type: isPasswordShows ? 'text' : 'password',
231:                 placeholder: t(TRANSLATION.PASSWORD),
232:               }}
233:               label={t(TRANSLATION.PASSWORD)}
234:               error={errors.password?.message}
```

### `src/components/modals/LoginModal/Login.tsx:223`
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
231:                 placeholder: t(TRANSLATION.PASSWORD),
232:               }}
233:               label={t(TRANSLATION.PASSWORD)}
234:               error={errors.password?.message}
235:               buttons={[
```

### `src/components/modals/LoginModal/Login.tsx:246`
```text
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
250:                       skipHandler: true,
251:                     } :
252:                     {}
253:                   ),
254:                 },
255:               ].filter(item => Object.values(item).length)}
256:             />
257:       }
258: 
```

### `src/components/modals/LoginModal/Login.tsx:248`
```text
242:                     {
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
257:       }
258: 
259:       <Checkbox
260:         {...formRegister('type')}
```

### `src/components/modals/LoginModal/Login.tsx:279`
```text
273: 
274:       {
275:         isVisible &&
276:             <div className="alert-container">
277:               <Alert
278:                 intent={status === EStatuses.Fail ? Intent.ERROR : Intent.SUCCESS}
279:                 message={(status === EStatuses.Fail ? t(TRANSLATION.LOGIN_FAIL) + ': ' + message : ( message === 'remind_password_success' ? t(TRANSLATION.REMIND_PASSWORD_SUCCESS) : t(TRANSLATION.LOGIN_SUCCESS)))}
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
```

### `src/components/modals/LoginModal/Login.tsx:286`
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
294:         //   // const obj = {
295:         //   //   u_name: data?.name,
296:         //   //   u_phone: '',
297:         //   //   u_email: 'moj14frffefff@gmail.com',          // TODO: заменить на data?.email
298:         //   //   type: ERegistrationType.Email,
```

### `src/components/modals/LoginModal/Login.tsx:288`
```text
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
294:         //   // const obj = {
295:         //   //   u_name: data?.name,
296:         //   //   u_phone: '',
297:         //   //   u_email: 'moj14frffefff@gmail.com',          // TODO: заменить на data?.email
298:         //   //   type: ERegistrationType.Email,
299:         //   //   u_role: EUserRoles.Client,
300:         //   //   ref_code: '',
```

### `src/components/modals/LoginModal/Login.tsx:304`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:311`
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
319:         fixedSize={false}
320:         className="login-modal_login-btn"
321:         skipHandler={true}
322:         disabled={!!Object.values(errors).length}
323:         status={status}
```

### `src/components/modals/LoginModal/Login.tsx:318`
```text
312:         </a>
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
327: }
328: 
329: export default connector(LoginForm)
```

### `src/components/modals/LoginModal/Login.tsx:320`
```text
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
327: }
328: 
329: export default connector(LoginForm)
```

### `src/components/modals/LoginModal/Login.tsx:329`
```text
323:         status={status}
324:       />
325:     </form>
326:   )
327: }
328: 
329: export default connector(LoginForm)
```

### `src/state/user/constants.ts:4`
```text
1: import { ArrayValue } from './../../types/index'
2: import { appName } from '../../constants'
3: import { TRANSLATION } from '../../localization'
4: import { EStatuses, IRegisterResponse, ITokens, IUser } from '../../types/types'
5: 
6: export const moduleName = 'user' as const
7: 
8: const prefix = `${appName}/${moduleName}`
9: 
10: export const ActionTypes = {
11:   REGISTER_REQUEST: `${prefix}/REGISTER_REQUEST`,
12:   REGISTER_START: `${prefix}/REGISTER_START`,
13:   REGISTER_SUCCESS: `${prefix}/REGISTER_SUCCESS`,
14:   REGISTER_FAIL: `${prefix}/REGISTER_FAIL`,
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
```

### `src/state/user/constants.ts:16`
```text
10: export const ActionTypes = {
11:   REGISTER_REQUEST: `${prefix}/REGISTER_REQUEST`,
12:   REGISTER_START: `${prefix}/REGISTER_START`,
13:   REGISTER_SUCCESS: `${prefix}/REGISTER_SUCCESS`,
14:   REGISTER_FAIL: `${prefix}/REGISTER_FAIL`,
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
```

### `src/state/user/constants.ts:17`
```text
11:   REGISTER_REQUEST: `${prefix}/REGISTER_REQUEST`,
12:   REGISTER_START: `${prefix}/REGISTER_START`,
13:   REGISTER_SUCCESS: `${prefix}/REGISTER_SUCCESS`,
14:   REGISTER_FAIL: `${prefix}/REGISTER_FAIL`,
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
```

### `src/state/user/constants.ts:18`
```text
12:   REGISTER_START: `${prefix}/REGISTER_START`,
13:   REGISTER_SUCCESS: `${prefix}/REGISTER_SUCCESS`,
14:   REGISTER_FAIL: `${prefix}/REGISTER_FAIL`,
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
```

### `src/state/user/constants.ts:19`
```text
13:   REGISTER_SUCCESS: `${prefix}/REGISTER_SUCCESS`,
14:   REGISTER_FAIL: `${prefix}/REGISTER_FAIL`,
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
```

### `src/state/user/constants.ts:21`
```text
15: 
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
```

### `src/state/user/constants.ts:22`
```text
16:   GOOGLE_LOGIN_REQUEST: `${prefix}/GOOGLE_LOGIN_REQUEST`,
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
```

### `src/state/user/constants.ts:23`
```text
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
```

### `src/state/user/constants.ts:24`
```text
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
```

### `src/state/user/constants.ts:25`
```text
19:   GOOGLE_LOGIN_FAIL: `${prefix}/GOOGLE_LOGIN_FAIL`,
20: 
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
```

### `src/state/user/constants.ts:27`
```text
21:   LOGIN_REQUEST: `${prefix}/LOGIN_REQUEST`,
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
```

### `src/state/user/constants.ts:28`
```text
22:   LOGIN_START: `${prefix}/LOGIN_START`,
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
40: 
```

### `src/state/user/constants.ts:29`
```text
23:   LOGIN_SUCCESS: `${prefix}/LOGIN_SUCCESS`,
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
40: 
41:   INIT_USER: `${prefix}/INIT_USER`,
```

### `src/state/user/constants.ts:30`
```text
24:   LOGIN_FAIL: `${prefix}/LOGIN_FAIL`,
25:   LOGIN_WHATSAPP: `${prefix}/LOGIN_WHATSAPP`,
26: 
27:   LOGOUT_REQUEST: `${prefix}/LOGOUT_REQUEST`,
28:   LOGOUT_START: `${prefix}/LOGOUT_START`,
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
40: 
41:   INIT_USER: `${prefix}/INIT_USER`,
42:   SET_USER: `${prefix}/SET_USER`,
```

### `src/state/user/constants.ts:43`
```text
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
40: 
41:   INIT_USER: `${prefix}/INIT_USER`,
42:   SET_USER: `${prefix}/SET_USER`,
43:   SET_TOKENS: `${prefix}/SET_TOKENS`,
44: 
45:   WHATSAPP_SIGNUP_SUCCESS: `${prefix}/WHATSAPP_SIGNUP_SUCCESS`,
46:   WHATSAPP_SIGNUP_START: `${prefix}/WHATSAPP_SIGNUP_START`,
47:   WHATSAPP_SIGNUP_FAIL: `${prefix}/WHATSAPP_SIGNUP_FAIL`,
48:   WHATSAPP_SIGNUP_REQUEST: `${prefix}/WHATSAPP_SIGNUP_REQUEST`,
49: } as const
50: 
51: export enum ERegistrationType {
52:   Email = 'e-mail',
53:   Phone = 'phone',
54:   Whatsapp = 'whatsapp',
55: }
```

### `src/state/user/constants.ts:57`
```text
51: export enum ERegistrationType {
52:   Email = 'e-mail',
53:   Phone = 'phone',
54:   Whatsapp = 'whatsapp',
55: }
56: 
57: export const LOGIN_TABS = [
58:   { label: TRANSLATION.SIGNIN, id: 'sign-in' },
59:   { label: TRANSLATION.SIGNUP, id: 'sign-up' },
60: ] as const
61: export const LOGIN_TABS_IDS = LOGIN_TABS.map(item => item.id)
62: 
63: export type TLoginTab = ArrayValue<typeof LOGIN_TABS_IDS>
64: 
65: export interface IUserState {
66:   user: IUser | null,
67:   tokens: ITokens | null,
68:   status: EStatuses,
69:   message: string,
```

### `src/state/user/constants.ts:61`
```text
55: }
56: 
57: export const LOGIN_TABS = [
58:   { label: TRANSLATION.SIGNIN, id: 'sign-in' },
59:   { label: TRANSLATION.SIGNUP, id: 'sign-up' },
60: ] as const
61: export const LOGIN_TABS_IDS = LOGIN_TABS.map(item => item.id)
62: 
63: export type TLoginTab = ArrayValue<typeof LOGIN_TABS_IDS>
64: 
65: export interface IUserState {
66:   user: IUser | null,
67:   tokens: ITokens | null,
68:   status: EStatuses,
69:   message: string,
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
73: }
```

### `src/state/user/constants.ts:63`
```text
57: export const LOGIN_TABS = [
58:   { label: TRANSLATION.SIGNIN, id: 'sign-in' },
59:   { label: TRANSLATION.SIGNUP, id: 'sign-up' },
60: ] as const
61: export const LOGIN_TABS_IDS = LOGIN_TABS.map(item => item.id)
62: 
63: export type TLoginTab = ArrayValue<typeof LOGIN_TABS_IDS>
64: 
65: export interface IUserState {
66:   user: IUser | null,
67:   tokens: ITokens | null,
68:   status: EStatuses,
69:   message: string,
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
73: }
```

### `src/state/user/constants.ts:67`
```text
61: export const LOGIN_TABS_IDS = LOGIN_TABS.map(item => item.id)
62: 
63: export type TLoginTab = ArrayValue<typeof LOGIN_TABS_IDS>
64: 
65: export interface IUserState {
66:   user: IUser | null,
67:   tokens: ITokens | null,
68:   status: EStatuses,
69:   message: string,
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
73: }
```

### `src/state/user/constants.ts:70`
```text
64: 
65: export interface IUserState {
66:   user: IUser | null,
67:   tokens: ITokens | null,
68:   status: EStatuses,
69:   message: string,
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
73: }
```

### `src/state/user/reducer.ts:1`
```text
1: import { ActionTypes, IUserState, LOGIN_TABS_IDS } from './constants'
2: import { Record } from 'immutable'
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
```

### `src/state/user/reducer.ts:9`
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
17: export default function reducer(state = new record(), action: TAction) {
18:   const { type, payload } = action
19: 
20:   switch (type) {
21:     case ActionTypes.LOGIN_START:
```

### `src/state/user/reducer.ts:12`
```text
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
17: export default function reducer(state = new record(), action: TAction) {
18:   const { type, payload } = action
19: 
20:   switch (type) {
21:     case ActionTypes.LOGIN_START:
22:       return state
23:         .set('status', EStatuses.Loading)
24:         .set('message', '')
```

### `src/state/user/reducer.ts:21`
```text
15: })
16: 
17: export default function reducer(state = new record(), action: TAction) {
18:   const { type, payload } = action
19: 
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

### `src/state/user/reducer.ts:26`
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
34:       return state
35:         .set('status', EStatuses.Fail)
36:         .set('message', payload)
37:     case ActionTypes.LOGIN_WHATSAPP:
38:       return state
```

### `src/state/user/reducer.ts:27`
```text
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
34:       return state
35:         .set('status', EStatuses.Fail)
36:         .set('message', payload)
37:     case ActionTypes.LOGIN_WHATSAPP:
38:       return state
39:         .set('status', EStatuses.Whatsapp)
```

### `src/state/user/reducer.ts:31`
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
39:         .set('status', EStatuses.Whatsapp)
40:         .set('message', 'Whatsapp message sent')
41: 
42:     case ActionTypes.GOOGLE_LOGIN_START:
43:       return state
```

### `src/state/user/reducer.ts:32`
```text
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
39:         .set('status', EStatuses.Whatsapp)
40:         .set('message', 'Whatsapp message sent')
41: 
42:     case ActionTypes.GOOGLE_LOGIN_START:
43:       return state
44:         .set('status', EStatuses.Loading)
```

### `src/state/user/reducer.ts:33`
```text
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
39:         .set('status', EStatuses.Whatsapp)
40:         .set('message', 'Whatsapp message sent')
41: 
42:     case ActionTypes.GOOGLE_LOGIN_START:
43:       return state
44:         .set('status', EStatuses.Loading)
45:         .set('message', '')
```

### `src/state/user/reducer.ts:37`
```text
31:         .set('tokens', payload.tokens)
32:     case ActionTypes.LOGIN_FAIL:
33:       console.log('LOGIN_FAIL', payload)
34:       return state
35:         .set('status', EStatuses.Fail)
36:         .set('message', payload)
37:     case ActionTypes.LOGIN_WHATSAPP:
38:       return state
39:         .set('status', EStatuses.Whatsapp)
40:         .set('message', 'Whatsapp message sent')
41: 
42:     case ActionTypes.GOOGLE_LOGIN_START:
43:       return state
44:         .set('status', EStatuses.Loading)
45:         .set('message', '')
46:         .set('user', null)
47:         .set('tokens', null)
48:     case ActionTypes.GOOGLE_LOGIN_SUCCESS:
49:       return state
```

### `src/state/user/reducer.ts:42`
```text
36:         .set('message', payload)
37:     case ActionTypes.LOGIN_WHATSAPP:
38:       return state
39:         .set('status', EStatuses.Whatsapp)
40:         .set('message', 'Whatsapp message sent')
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

### `src/state/user/reducer.ts:47`
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
55:         .set('status', EStatuses.Fail)
56:         .set('message', TRANSLATION.LOGIN_FAIL)
57: 
58:     case ActionTypes.LOGOUT_START:
59:       return state
```

### `src/state/user/reducer.ts:48`
```text
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
55:         .set('status', EStatuses.Fail)
56:         .set('message', TRANSLATION.LOGIN_FAIL)
57: 
58:     case ActionTypes.LOGOUT_START:
59:       return state
60:         .set('status', EStatuses.Loading)
```

### `src/state/user/reducer.ts:52`
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
60:         .set('status', EStatuses.Loading)
61:         .set('message', '')
62:         .set('user', null)
63:         .set('tokens', null)
64:     case ActionTypes.LOGOUT_SUCCESS:
```

### `src/state/user/reducer.ts:53`
```text
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
60:         .set('status', EStatuses.Loading)
61:         .set('message', '')
62:         .set('user', null)
63:         .set('tokens', null)
64:     case ActionTypes.LOGOUT_SUCCESS:
65:       return state
```

### `src/state/user/reducer.ts:56`
```text
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
60:         .set('status', EStatuses.Loading)
61:         .set('message', '')
62:         .set('user', null)
63:         .set('tokens', null)
64:     case ActionTypes.LOGOUT_SUCCESS:
65:       return state
66:         .set('status', EStatuses.Success)
67:         .set('message', TRANSLATION.LOGOUT_SUCCESS)
68:     case ActionTypes.LOGOUT_FAIL:
```

### `src/state/user/reducer.ts:58`
```text
52:         .set('tokens', payload.tokens)
53:     case ActionTypes.GOOGLE_LOGIN_FAIL:
54:       return state
55:         .set('status', EStatuses.Fail)
56:         .set('message', TRANSLATION.LOGIN_FAIL)
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

### `src/state/user/reducer.ts:63`
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
71:         .set('message', TRANSLATION.LOGOUT_FAIL)
72: 
73:     case ActionTypes.REGISTER_START:
74:       return state
75:         .set('status', EStatuses.Loading)
```

### `src/state/user/reducer.ts:64`
```text
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
71:         .set('message', TRANSLATION.LOGOUT_FAIL)
72: 
73:     case ActionTypes.REGISTER_START:
74:       return state
75:         .set('status', EStatuses.Loading)
76:         .set('message', '')
```

### `src/state/user/reducer.ts:67`
```text
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
71:         .set('message', TRANSLATION.LOGOUT_FAIL)
72: 
73:     case ActionTypes.REGISTER_START:
74:       return state
75:         .set('status', EStatuses.Loading)
76:         .set('message', '')
77:         .set('user', null)
78:         .set('tokens', null)
79:     case ActionTypes.REGISTER_SUCCESS:
```

### `src/state/user/reducer.ts:68`
```text
62:         .set('user', null)
63:         .set('tokens', null)
64:     case ActionTypes.LOGOUT_SUCCESS:
65:       return state
66:         .set('status', EStatuses.Success)
67:         .set('message', TRANSLATION.LOGOUT_SUCCESS)
68:     case ActionTypes.LOGOUT_FAIL:
69:       return state
70:         .set('status', EStatuses.Fail)
71:         .set('message', TRANSLATION.LOGOUT_FAIL)
72: 
73:     case ActionTypes.REGISTER_START:
74:       return state
75:         .set('status', EStatuses.Loading)
76:         .set('message', '')
77:         .set('user', null)
78:         .set('tokens', null)
79:     case ActionTypes.REGISTER_SUCCESS:
80:       return state
```

### `src/state/user/reducer.ts:71`
```text
65:       return state
66:         .set('status', EStatuses.Success)
67:         .set('message', TRANSLATION.LOGOUT_SUCCESS)
68:     case ActionTypes.LOGOUT_FAIL:
69:       return state
70:         .set('status', EStatuses.Fail)
71:         .set('message', TRANSLATION.LOGOUT_FAIL)
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
```

### `src/state/user/reducer.ts:78`
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
86:         .set('status', EStatuses.Fail)
87:         .set('message', payload && payload.message || TRANSLATION.REGISTER_FAIL)
88: 
89:     case ActionTypes.REMIND_PASSWORD_START:
90:       return state
```

### `src/state/user/reducer.ts:94`
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
102:         .set('message', TRANSLATION.REMIND_PASSWORD_FAIL)
103: 
104:     case ActionTypes.SET_TAB:
105:       return state
106:         .set('tab', payload)
```

### `src/state/user/reducer.ts:109`
```text
103: 
104:     case ActionTypes.SET_TAB:
105:       return state
106:         .set('tab', payload)
107:         .set('status', EStatuses.Default)
108:         .set('message', '')
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
```

### `src/state/user/reducer.ts:111`
```text
105:       return state
106:         .set('tab', payload)
107:         .set('status', EStatuses.Default)
108:         .set('message', '')
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
123:       return state
```

### `src/state/user/reducer.ts:115`
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
123:       return state
124:         .set('status', EStatuses.Success)
125:         .set('message', TRANSLATION.REGISTER_SUCCESS)
126:         .set('response', payload)
127:         .set('whatsappSignUpData', null)
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
```

### `src/tools/api.ts:23`
```text
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
```

### `src/tools/api.ts:26`
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
```

### `src/tools/api.ts:27`
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
```

### `src/tools/api.ts:29`
```text
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
```

### `src/tools/api.ts:32`
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
```

### `src/tools/api.ts:33`
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
```

### `src/tools/api.ts:38`
```text
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
```

### `src/tools/api.ts:39`
```text
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
```

### `src/components/modals/LoginModal/index.tsx:10`
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
18: import RegisterForm from './Register'
19: import LogoutForm from './LogoutForm'
20: import RegisterJSON from './RegisterJSON'
21: import './styles.scss'
22: 
```

### `src/components/modals/LoginModal/index.tsx:17`
```text
11: import { modalsActionCreators, modalsSelectors } from '../../../state/modals'
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
26:   status: userSelectors.status(state),
27:   tab: userSelectors.tab(state),
28: })
29: 
```

### `src/components/modals/LoginModal/index.tsx:19`
```text
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
26:   status: userSelectors.status(state),
27:   tab: userSelectors.tab(state),
28: })
29: 
30: const mapDispatchToProps = {
31:   setLoginModal: modalsActionCreators.setLoginModal,
```

### `src/components/modals/LoginModal/index.tsx:24`
```text
18: import RegisterForm from './Register'
19: import LogoutForm from './LogoutForm'
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
34: 
35: const connector = connect(mapStateToProps, mapDispatchToProps)
36: 
```

### `src/components/modals/LoginModal/index.tsx:31`
```text
25:   user: userSelectors.user(state),
26:   status: userSelectors.status(state),
27:   tab: userSelectors.tab(state),
28: })
29: 
30: const mapDispatchToProps = {
31:   setLoginModal: modalsActionCreators.setLoginModal,
32:   setTab: userActionCreators.setTab,
33: }
34: 
35: const connector = connect(mapStateToProps, mapDispatchToProps)
36: 
37: interface IProps extends ConnectedProps<typeof connector> {}
38: 
39: function LoginModal({
40:   isOpen,
41:   user,
42:   status,
43:   tab,
```

### `src/components/modals/LoginModal/index.tsx:39`
```text
33: }
34: 
35: const connector = connect(mapStateToProps, mapDispatchToProps)
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
50:     LOGIN_TABS.map((item, index) => ({
51:       ...item,
```

### `src/components/modals/LoginModal/index.tsx:45`
```text
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
50:     LOGIN_TABS.map((item, index) => ({
51:       ...item,
52:       label: t(item.label),
53:     }))
54: 
55:   const RegisterComponent = location.pathname.includes('/driver-order') ?
56:     RegisterJSON :
57:     RegisterForm
```

### `src/components/modals/LoginModal/index.tsx:50`
```text
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
57:     RegisterForm
58:   return (
59:     <Overlay
60:       isOpen={isOpen}
61:       onClick={() => setLoginModal(false)}
62:     >
```

### `src/components/modals/LoginModal/index.tsx:61`
```text
55:   const RegisterComponent = location.pathname.includes('/driver-order') ?
56:     RegisterJSON :
57:     RegisterForm
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
72:                 t(TRANSLATION.SIGNUP)
73:             }
```

### `src/components/modals/LoginModal/index.tsx:63`
```text
57:     RegisterForm
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
72:                 t(TRANSLATION.SIGNUP)
73:             }
74:           </legend>
75: 
```

### `src/components/modals/LoginModal/index.tsx:76`
```text
70:               tab === 'sign-in' ?
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
85:             {useMemo(() => user ?
86:               <LogoutForm /> :
87:               tab === 'sign-in' ?
88:                 <LoginForm
```

### `src/components/modals/LoginModal/index.tsx:86`
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
94:             , [user, tab, isOpen])}
95:           </div>
96:         </fieldset>
97:         <VersionInfo/>
98:       </div>
```

### `src/components/modals/LoginModal/index.tsx:88`
```text
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
94:             , [user, tab, isOpen])}
95:           </div>
96:         </fieldset>
97:         <VersionInfo/>
98:       </div>
99:     </Overlay>
100:   )
```

### `src/components/modals/LoginModal/index.tsx:103`
```text
97:         <VersionInfo/>
98:       </div>
99:     </Overlay>
100:   )
101: }
102: 
103: export default connector(LoginModal)
```

### `src/API/user.ts:27`
```text
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
```

### `src/API/user.ts:30`
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
```

### `src/API/user.ts:34`
```text
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
```

### `src/API/user.ts:36`
```text
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
```

### `src/API/user.ts:93`
```text
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
```

### `src/API/user.ts:98`
```text
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

### `src/components/modals/LoginModal/RefCodeModal.tsx:35`
```text
29:   user: userSelectors.user(state),
30:   whatsappSignUpData: userSelectors.whatsappSignUpData(state),
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
45: }
46: 
47: const RefCodeModal: React.FC<IProps> = ({
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:36`
```text
30:   whatsappSignUpData: userSelectors.whatsappSignUpData(state),
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
45: }
46: 
47: const RefCodeModal: React.FC<IProps> = ({
48:   payload,
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:50`
```text
44: interface IProps extends ConnectedProps<typeof connector> {
45: }
46: 
47: const RefCodeModal: React.FC<IProps> = ({
48:   payload,
49:   setRefCodeModal,
50:   googleLogin,
51:   status,
52:   whatsappSignUp,
53:   whatsappSignUpData,
54: }) => {
55:   const [isVisible, toggleVisibility] = useVisibility(false)
56: 
57:   const navigate = useNavigate()
58: 
59:   const schema = yup.object({
60:     code: yup.string(),
61:   })
62:   useEffect(() => {
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:91`
```text
85:   const onSubmit = (formData : IFormValues) => {
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
100: 
101:     API.checkRefCode(formData.code).then(isFreeCode => {
102:       if (isFreeCode) {
103:         setError('code', { type: 'custom', message: t(TRANSLATION.REF_CODE_NOT_FOUND) })
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:96`
```text
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
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:111`
```text
105:       }
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
120:     })
121:   }
122: 
123:   return (
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:117`
```text
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
```

## 3. Evidence discipline

На этом проходе не утверждается, что все найденные `token`, `u_hash`, `id_role` относятся к одному auth flow.

Каждый Claim должен быть привязан к конкретному path:

```text
auth action
  → response
  → state/storage
  → request adapter```

## 4. Требуемая связь с Core Backend

Backend RP-20 установил общий authentication gate:

```text
API request
   ↓
check_auth_user()
   ↓
authenticated context```

Следующая задача — установить, какие frontend credentials фактически удовлетворяют этому gate.

## 5. Current semantic boundary

Пока допустимы только такие отдельные Claims:

```text
Taxi Frontend    → HAS_AUTH_IMPLEMENTATION
Frontend API layer    → CARRIES_AUTH_DATA
Core Backend    → AUTHENTICATES_REQUEST```

Но пока не добавляем без прямого value-flow:

```text
Frontend token    → IS_SAME_AS    → Backend session credential```

## 6. Snapshot boundary

Все Claims этого документа принадлежат:

```text
Frontend Snapshot = taxi-master.zip```

Они не становятся свойствами Core Backend и не распространяются автоматически на другие frontend clients.

## 7. Gap Report

```text
G-32-01  exact auth endpoint/request              OPEN
G-32-02  exact response credential fields          OPEN
G-32-03  credential persistence/state              OPEN
G-32-04  API adapter credential injection           OPEN
G-32-05  frontend credential → check_auth_user      OPEN
```

## 8. Следующий шаг

Нужно выбрать один конкретный auth endpoint из найденных contexts и пройти его полностью:

```text
Frontend auth call
      ↓
request fields
      ↓
backend auth response
      ↓
token / u_hash / identity
      ↓
storage/state
      ↓
one ordinary API request
      ↓
credential injection```

Только после этого можно зафиксировать полноценный Frontend ↔ Core Backend Authentication relation.