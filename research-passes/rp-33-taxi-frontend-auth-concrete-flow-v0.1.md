# Backend Semantic Graph — Research Pass 33
# Taxi Frontend Authentication Concrete Flow v0.1

**Статус:** EVIDENCE-GROUNDED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-32 Taxi Frontend Authentication API → State v0.1  
**Источник:** `taxi-master.zip` — конкретный frontend snapshot

## 1. Research Question

> Какой конкретный frontend authentication call формирует authenticated API context и какие данные после него используются обычными API requests?

Цель — закрыть одну полную цепочку, а не каталогизировать все `token`/`u_hash` occurrences.

## 2. Concrete auth-related source contexts

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
```

### `src/API/auth.ts:53`
```text
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
```

### `src/API/auth.ts:56`
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
```

### `src/API/auth.ts:60`
```text
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
```

### `src/API/auth.ts:69`
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
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
```

### `src/API/auth.ts:72`
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

### `src/API/auth.ts:79`
```text
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
```

### `src/API/auth.ts:86`
```text
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
```

### `src/API/auth.ts:93`
```text
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
```

### `src/API/auth.ts:101`
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
```

### `src/API/auth.ts:102`
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
```

### `src/API/auth.ts:105`
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
```

### `src/API/auth.ts:106`
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
```

### `src/API/auth.ts:107`
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
```

### `src/API/auth.ts:109`
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
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
```

### `src/API/auth.ts:110`
```text
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
```

### `src/API/auth.ts:111`
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
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
```

### `src/API/auth.ts:117`
```text
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

### `src/API/auth.ts:122`
```text
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

### `src/API/auth.ts:129`
```text
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
```

### `src/API/auth.ts:142`
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
```

### `src/API/auth.ts:157`
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
```

### `src/API/auth.ts:166`
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
```

### `src/API/auth.ts:169`
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

### `src/API/auth.ts:170`
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
```

### `src/API/auth.ts:171`
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
182:               },
183:             }
184:           })
185:       })
```

### `src/API/auth.ts:172`
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
183:             }
184:           })
185:       })
186:   }
```

### `src/API/auth.ts:174`
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
```

### `src/API/auth.ts:179`
```text
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

### `src/API/auth.ts:180`
```text
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

### `src/API/auth.ts:181`
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
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
```

### `src/API/auth.ts:188`
```text
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

### `src/API/auth.ts:189`
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
```

### `src/API/auth.ts:190`
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
```

### `src/API/auth.ts:191`
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
```

### `src/API/auth.ts:193`
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
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
```

### `src/API/auth.ts:194`
```text
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

### `src/API/auth.ts:195`
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
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:196`
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
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:202`
```text
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

### `src/API/car.ts:95`
```text
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
```

### `src/API/car.ts:184`
```text
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

### `src/API/index.ts:33`
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
```

### `src/API/index.ts:35`
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
```

### `src/API/index.ts:41`
```text
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
```

### `src/API/index.ts:85`
```text
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
```

### `src/API/index.ts:86`
```text
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
```

### `src/API/index.ts:288`
```text
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
```

### `src/API/index.ts:297`
```text
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
```

### `src/API/user.ts:27`
```text
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
```

### `src/API/user.ts:30`
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
```

### `src/API/user.ts:34`
```text
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
```

### `src/API/user.ts:36`
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
47:   'ref_code' |
48:   'u_details'
49: >>
50: export type TEditDriverCheckRequired = TEditUser & Partial<Pick<IUser,
```

### `src/API/user.ts:93`
```text
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
```

### `src/API/user.ts:98`
```text
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

### `src/components/modals/LoginModal/Login.tsx:12`
```text
4: import Checkbox from '../../Checkbox'
5: import { useForm, useWatch } from 'react-hook-form'
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
25: const mapStateToProps = (state: IRootState) => ({
26:   user: userSelectors.user(state),
```

### `src/components/modals/LoginModal/Login.tsx:19`
```text
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
25: const mapStateToProps = (state: IRootState) => ({
26:   user: userSelectors.user(state),
27:   status: userSelectors.status(state),
28:   tab: userSelectors.tab(state),
29:   message: userSelectors.message(state),
30:   isWAOpen: modalsSelectors.isWACodeModalOpen,
31: })
32: 
33: const mapDispatchToProps = {
```

### `src/components/modals/LoginModal/Login.tsx:34`
```text
26:   user: userSelectors.user(state),
27:   status: userSelectors.status(state),
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
47: 
48: interface IFormValues {
```

### `src/components/modals/LoginModal/Login.tsx:35`
```text
27:   status: userSelectors.status(state),
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
47: 
48: interface IFormValues {
49:     login: string | undefined,
```

### `src/components/modals/LoginModal/Login.tsx:36`
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
47: 
48: interface IFormValues {
49:     login: string | undefined,
50:     password?: string | undefined,
```

### `src/components/modals/LoginModal/Login.tsx:49`
```text
41:   register: userActionCreators.register,
42:   setWAOpen: modalsActionCreators.setWACodeModal,
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
62:   tab,
63:   googleLogin,
```

### `src/components/modals/LoginModal/Login.tsx:59`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:63`
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
69:   remindPassword,
70:   setMessage,
71:   setStatus,
72:   setLoginModal,
73:   message,
74: }) => {
75:   const [isPasswordShows, setIsPasswordShows] = useState(false)
76:   const [dataToLogin, setDataToLogin] = useState({})
77:   const [isVisible, toggleVisibility] = useVisibility(false)
```

### `src/components/modals/LoginModal/Login.tsx:67`
```text
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
76:   const [dataToLogin, setDataToLogin] = useState({})
77:   const [isVisible, toggleVisibility] = useVisibility(false)
78:   const [isPasswordVisible, togglePasswordVisibility] = useVisibility(true)
79:   const location = useLocation()
80:   const navigate = useNavigate()
81:   const googleClientId = '973943716904-b33r11ijgi08m5etsg5ndv409shh1tjl.apps.googleusercontent.com'
```

### `src/components/modals/LoginModal/Login.tsx:72`
```text
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
81:   const googleClientId = '973943716904-b33r11ijgi08m5etsg5ndv409shh1tjl.apps.googleusercontent.com'
82: 
83:   const role = !location.pathname.includes('/driver-order') ?
84:     EUserRoles.Client :
85:     EUserRoles.Driver
86: 
```

### `src/components/modals/LoginModal/Login.tsx:76`
```text
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
85:     EUserRoles.Driver
86: 
87:   const schema = yup.object({
88:     type: yup.string().required(),
89:     login: yup.string().required().when('type', {
90:       is: (type: ERegistrationType) => (type === ERegistrationType.Email),
```

### `src/components/modals/LoginModal/Login.tsx:89`
```text
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
102:     trigger,
103:   } = useForm<IFormValues>({
```

### `src/components/modals/LoginModal/Login.tsx:107`
```text
99:     handleSubmit,
100:     formState: { errors, isDirty },
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
120:       toggleVisibility()
121:     }
```

### `src/components/modals/LoginModal/Login.tsx:112`
```text
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
120:       toggleVisibility()
121:     }
122:   }, [isOpen])
123: 
124:   useEffect(() => {
125:     let u_email = getParamFromURL('u_email')
126:     let u_name = getParamFromURL('u_name')
```

### `src/components/modals/LoginModal/Login.tsx:166`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:169`
```text
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
179:       }
180:     }
181:   }, [status])
182: 
183:   if (tab !== LOGIN_TABS_IDS[0]) return null
```

### `src/components/modals/LoginModal/Login.tsx:170`
```text
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
179:       }
180:     }
181:   }, [status])
182: 
183:   if (tab !== LOGIN_TABS_IDS[0]) return null
184: 
```

### `src/components/modals/LoginModal/Login.tsx:183`
```text
175:     if(status === EStatuses.Success && message==='remind_password_success') {
176:       toggleVisibility()
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
196:         } :
197:         {
```

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
```

### `src/components/modals/LoginModal/Login.tsx:192`
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
203:   }
204: 
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
```

### `src/components/modals/LoginModal/Login.tsx:195`
```text
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
205:   const getParamFromURL = (param: string) => {
206:     const value = new URLSearchParams(location.search).get(param)
207:     return value && decodeURIComponent(value)
208:   }
209: 
```

### `src/components/modals/LoginModal/Login.tsx:199`
```text
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
208:   }
209: 
210:   return (
211:     <form
212:       className="login-form sign-in-subform"
213:       onSubmit={handleSubmit(onSubmit)}
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
```

### `src/components/modals/LoginModal/Login.tsx:212`
```text
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
```

### `src/components/modals/LoginModal/Login.tsx:217`
```text
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
225:       />
226:       {isPasswordVisible &&
227:             <Input
228:               inputProps={{
229:                 ...formRegister('password'),
230:                 type: isPasswordShows ? 'text' : 'password',
231:                 placeholder: t(TRANSLATION.PASSWORD),
```

### `src/components/modals/LoginModal/Login.tsx:222`
```text
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
230:                 type: isPasswordShows ? 'text' : 'password',
231:                 placeholder: t(TRANSLATION.PASSWORD),
232:               }}
233:               label={t(TRANSLATION.PASSWORD)}
234:               error={errors.password?.message}
235:               buttons={[
236:                 {
```

### `src/components/modals/LoginModal/Login.tsx:223`
```text
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
230:                 type: isPasswordShows ? 'text' : 'password',
231:                 placeholder: t(TRANSLATION.PASSWORD),
232:               }}
233:               label={t(TRANSLATION.PASSWORD)}
234:               error={errors.password?.message}
235:               buttons={[
236:                 {
237:                   src: isPasswordShows ? images.closedEye : images.openedEye,
```

### `src/components/modals/LoginModal/Login.tsx:246`
```text
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

### `src/components/modals/LoginModal/Login.tsx:248`
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
259:       <Checkbox
260:         {...formRegister('type')}
261:         type="radio"
262:         label={t(TRANSLATION.EMAIL)}
```

### `src/components/modals/LoginModal/Login.tsx:279`
```text
271:         id="whatsapp"
272:       />
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
292:         //   onResolve={(data) => {
293:         //     console.log(data)
```

### `src/components/modals/LoginModal/Login.tsx:286`
```text
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

### `src/components/modals/LoginModal/Login.tsx:288`
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
299:         //   //   u_role: EUserRoles.Client,
300:         //   //   ref_code: '',
301:         //   //   u_details: {},
302:         //   //   st: '1',
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
```

### `src/components/modals/LoginModal/Login.tsx:311`
```text
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
320:         className="login-modal_login-btn"
321:         skipHandler={true}
322:         disabled={!!Object.values(errors).length}
323:         status={status}
324:       />
325:     </form>
```

### `src/components/modals/LoginModal/Login.tsx:320`
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

### `src/components/modals/LoginModal/Login.tsx:329`
```text
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

### `src/components/modals/LoginModal/LogoutForm.tsx:32`
```text
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
```

### `src/components/modals/LoginModal/LogoutForm.tsx:43`
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
49:         fixedSize={false}
50:         className="login-modal_login-btn"
51:         skipHandler={true}
52:         onClick={() => {
53:           logout()
54:         }}
55:       />
56:     </form>
57:   )
```

### `src/components/modals/LoginModal/LogoutForm.tsx:50`
```text
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

### `src/components/modals/LoginModal/RefCodeModal.tsx:35`
```text
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
49:   setRefCodeModal,
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:36`
```text
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
49:   setRefCodeModal,
50:   googleLogin,
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:50`
```text
42: const connector = connect(mapStateToProps, mapDispatchToProps)
43: 
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
63:     if(status === EStatuses.Fail && !isVisible) {
64:       toggleVisibility()
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:91`
```text
83:   })
84: 
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
104:         return
105:       }
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
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:111`
```text
103:         setError('code', { type: 'custom', message: t(TRANSLATION.REF_CODE_NOT_FOUND) })
104:         return
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
124:     <Overlay
125:       isOpen={payload.isOpen}
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
```

### `src/components/modals/LoginModal/RefCodeModal.tsx:146`
```text
138:                 }}
139:               />
140: 
141:               {
142:                 isVisible &&
143:                   <div className="alert-container">
144:                     <Alert
145:                       intent={Intent.ERROR}
146:                       message={t(TRANSLATION.LOGIN_FAIL)}
147:                       onClose={toggleVisibility}
148:                     />
149:                   </div>
150:               }
151: 
152:               <Button
153:                 type={'submit'}
154:                 skipHandler={true}
155:                 disabled={!!Object.values(errors).length}
156:                 text={t(TRANSLATION.SIGNUP)}
157:                 className="refcode-modal-btn"
158:               />
159:             </div>
160:           </fieldset>
```

### `src/components/modals/LoginModal/Register.tsx:12`
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
18: import { yupResolver } from '@hookform/resolvers/yup'
19: import * as yup from 'yup'
20: import Alert from '../../Alert/Alert'
21: import { Intent } from '../../Alert'
22: import { useVisibility } from '../../../tools/hooks'
23: import { ISelectOption } from '../../../types'
24: import { normalizePhoneNumber } from '../../../tools/phoneUtils'
25: import { Input } from './elements'
26: 
```

### `src/components/modals/LoginModal/Register.tsx:234`
```text
226:     if (type !== ERegistrationType.Email) {
227:       setValue('u_email', '')
228:     }
229:     if (type !== ERegistrationType.Phone) {
230:       setValue('u_phone', '')
231:     }
232:   }, [type])
233: 
234:   if (tab !== LOGIN_TABS_IDS[1]) return null
235: 
236:   const onSubmit = (data: IFormValues) => {
237:     if (getPhoneError(u_phone, type === ERegistrationType.Phone)) return
238: 
239:     if (isRegistrationAlertVisible) {
240:       toggleRegistrationAlertVisibility()
241:     }
242: 
243:     let upload: any[] = []
244:     if (filesMap.passport_photo) {
245:       Array.from(filesMap.passport_photo).forEach(file => {
246:         upload.push({ name: 'passport_photo', file })
247:       })
248:     }
```

### `src/components/modals/LoginModal/Register.tsx:328`
```text
320:   }
321: 
322:   const isDriver = Number(u_role) === EUserRoles.Driver
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
337:             { label: t(TRANSLATION.SELF_EMPLOYED), value: EWorkTypes.Self },
338:             { label: t(TRANSLATION.COMPANY), value: EWorkTypes.Company },
339:           ]}
340:         />
341:       }
342: 
```

### `src/components/modals/LoginModal/Register.tsx:636`
```text
628:           />
629:         </div>
630:       )}
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
645: export default connector(RegisterForm)
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
```

### `src/components/modals/LoginModal/WACodeModal.tsx:113`
```text
105:                 label={t(TRANSLATION.CODE_WRITE)}
106:               />
107: 
108:               {
109:                 isVisible &&
110:                   <div className="alert-container">
111:                     <Alert
112:                       intent={Intent.ERROR}
113:                       message={t(TRANSLATION.LOGIN_FAIL)}
114:                       onClose={toggleVisibility}
115:                     />
116:                   </div>
117:               }
118: 
119:               <Button
120:                 type={'submit'}
121:                 skipHandler={true}
122:                 disabled={!!Object.values(errors).length}
123:                 text={t(TRANSLATION.SIGN_IN)}
124:                 className="whatsapp-modal-btn"
125:               />
126:             </div>
127:           </fieldset>
```

### `src/components/modals/LoginModal/elements.tsx:11`
```text
3: import BaseInput, { EInputStyles } from '../../Input'
4: 
5: export function Input({
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
4: 
5: export function Input({
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

### `src/components/modals/LoginModal/index.tsx:10`
```text
2: import { connect, ConnectedProps } from 'react-redux'
3: import { useLocation } from 'react-router-dom'
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
23: const mapStateToProps = (state: IRootState) => ({
24:   isOpen: modalsSelectors.isLoginModalOpen(state),
```

### `src/components/modals/LoginModal/index.tsx:17`
```text
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
32:   setTab: userActionCreators.setTab,
33: }
34: 
35: const connector = connect(mapStateToProps, mapDispatchToProps)
36: 
37: interface IProps extends ConnectedProps<typeof connector> {}
38: 
```

### `src/components/modals/LoginModal/index.tsx:31`
```text
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
37: interface IProps extends ConnectedProps<typeof connector> {}
38: 
39: function LoginModal({
40:   isOpen,
41:   user,
42:   status,
43:   tab,
44:   setTab,
45:   setLoginModal,
```

### `src/components/modals/LoginModal/index.tsx:39`
```text
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
```

### `src/components/modals/LoginModal/index.tsx:45`
```text
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
52:       label: t(item.label),
53:     }))
54: 
55:   const RegisterComponent = location.pathname.includes('/driver-order') ?
56:     RegisterJSON :
57:     RegisterForm
58:   return (
59:     <Overlay
```

### `src/components/modals/LoginModal/index.tsx:50`
```text
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
58:   return (
59:     <Overlay
60:       isOpen={isOpen}
61:       onClick={() => setLoginModal(false)}
62:     >
63:       <div className="modal login-modal">
64:         {status === EStatuses.Loading && <LoadFrame />}
```

### `src/components/modals/LoginModal/index.tsx:61`
```text
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

### `src/components/modals/LoginModal/index.tsx:63`
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
74:           </legend>
75: 
76:           <div className="login">
77:             {useMemo(() => _TABS.length > 0 &&
```

### `src/components/modals/LoginModal/index.tsx:76`
```text
68:             {user ?
69:               t(TRANSLATION.PROFILE) :
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
89:                   isOpen={isOpen}
90:                 /> :
```

### `src/components/modals/LoginModal/index.tsx:88`
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
99:     </Overlay>
100:   )
101: }
102: 
```

### `src/components/modals/LoginModal/index.tsx:103`
```text
95:           </div>
96:         </fieldset>
97:         <VersionInfo/>
98:       </div>
99:     </Overlay>
100:   )
101: }
102: 
103: export default connector(LoginModal)
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
```

### `src/tools/api.ts:23`
```text
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

### `src/tools/api.ts:26`
```text
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
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
```

### `src/tools/api.ts:27`
```text
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
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
```

### `src/tools/api.ts:29`
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
```

### `src/tools/api.ts:32`
```text
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
```

### `src/tools/api.ts:33`
```text
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
```

### `src/tools/api.ts:38`
```text
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
```

### `src/tools/api.ts:39`
```text
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
```

### `src/components/Button/index.tsx:17`
```text
9: import { getStatusClassName } from '../../tools/utils'
10: import { gradient } from '../../tools/theme'
11: 
12: const mapStateToProps = (state: IRootState) => ({
13:   user: userSelectors.user(state),
14: })
15: 
16: const mapDispatchToProps = {
17:   setLoginModal: modalsActionCreators.setLoginModal,
18: }
19: 
20: const connector = connect(mapStateToProps, mapDispatchToProps)
21: 
22: export enum EButtonShapes {
23:   Default,
24:   Flat,
25: }
26: 
27: export enum EButtonStyles {
28:   Default,
29:   RedDesign,
30: }
31: 
```

### `src/components/Button/index.tsx:45`
```text
37:   shape?: EButtonShapes,
38:   buttonStyle?: EButtonStyles,
39:   skipHandler?: boolean,
40:   text?: string,
41:   svg?: ReactElement,
42:   label?: string,
43:   status?: EStatuses,
44:   colorType?: EColorTypes,
45:   checkLogin?: boolean
46: }
47: 
48: function Button({
49:   wrapperProps = {},
50:   imageProps,
51:   fixedSize = true,
52:   shape = EButtonShapes.Default,
53:   buttonStyle = EButtonStyles.Default,
54:   skipHandler,
55:   text,
56:   svg,
57:   label,
58:   status = EStatuses.Default,
59:   user,
```

### `src/components/Button/index.tsx:60`
```text
52:   shape = EButtonShapes.Default,
53:   buttonStyle = EButtonStyles.Default,
54:   skipHandler,
55:   text,
56:   svg,
57:   label,
58:   status = EStatuses.Default,
59:   user,
60:   setLoginModal,
61:   colorType = EColorTypes.Default,
62:   checkLogin = true,
63:   ...buttonProps
64: }: IProps) {
65:   const handleButtonClick = useCallback((
66:     e: React.PointerEvent<HTMLButtonElement>,
67:   ): void => {
68:     if (skipHandler) return buttonProps.onClick && buttonProps.onClick(e)
69:     const loggedIn = !checkLogin || user
70: 
71:     if (
72:       buttonProps.type !== 'submit' ||
73:       (buttonProps.type === 'submit' && !loggedIn)
74:     ) {
```

### `src/components/Button/index.tsx:62`
```text
54:   skipHandler,
55:   text,
56:   svg,
57:   label,
58:   status = EStatuses.Default,
59:   user,
60:   setLoginModal,
61:   colorType = EColorTypes.Default,
62:   checkLogin = true,
63:   ...buttonProps
64: }: IProps) {
65:   const handleButtonClick = useCallback((
66:     e: React.PointerEvent<HTMLButtonElement>,
67:   ): void => {
68:     if (skipHandler) return buttonProps.onClick && buttonProps.onClick(e)
69:     const loggedIn = !checkLogin || user
70: 
71:     if (
72:       buttonProps.type !== 'submit' ||
73:       (buttonProps.type === 'submit' && !loggedIn)
74:     ) {
75:       e.preventDefault()
76:       e.stopPropagation()
```

### `src/components/Button/index.tsx:69`
```text
61:   colorType = EColorTypes.Default,
62:   checkLogin = true,
63:   ...buttonProps
64: }: IProps) {
65:   const handleButtonClick = useCallback((
66:     e: React.PointerEvent<HTMLButtonElement>,
67:   ): void => {
68:     if (skipHandler) return buttonProps.onClick && buttonProps.onClick(e)
69:     const loggedIn = !checkLogin || user
70: 
71:     if (
72:       buttonProps.type !== 'submit' ||
73:       (buttonProps.type === 'submit' && !loggedIn)
74:     ) {
75:       e.preventDefault()
76:       e.stopPropagation()
77:     }
78: 
79:     if (loggedIn) {
80:       if (buttonProps.onClick) buttonProps.onClick(e)
81:     } else {
82:       setLoginModal(true)
83:     }
```

### `src/components/Button/index.tsx:82`
```text
74:     ) {
75:       e.preventDefault()
76:       e.stopPropagation()
77:     }
78: 
79:     if (loggedIn) {
80:       if (buttonProps.onClick) buttonProps.onClick(e)
81:     } else {
82:       setLoginModal(true)
83:     }
84:   }, [
85:     buttonProps.type, buttonProps.onClick,
86:     skipHandler, checkLogin, user, setLoginModal,
87:   ])
88: 
89:   return (
90:     <>
91:       {label && <span className={`button__label button__label--${getStatusClassName(status)}`}>{label}</span>}
92:       <div {...wrapperProps} className={cn('button__wrapper', wrapperProps.className)}>
93:         <button
94:           {...buttonProps}
95:           className={cn(
96:             'button',
```

### `src/components/Button/index.tsx:86`
```text
78: 
79:     if (loggedIn) {
80:       if (buttonProps.onClick) buttonProps.onClick(e)
81:     } else {
82:       setLoginModal(true)
83:     }
84:   }, [
85:     buttonProps.type, buttonProps.onClick,
86:     skipHandler, checkLogin, user, setLoginModal,
87:   ])
88: 
89:   return (
90:     <>
91:       {label && <span className={`button__label button__label--${getStatusClassName(status)}`}>{label}</span>}
92:       <div {...wrapperProps} className={cn('button__wrapper', wrapperProps.className)}>
93:         <button
94:           {...buttonProps}
95:           className={cn(
96:             'button',
97:             { disabled: buttonProps.disabled },
98:             { 'button--accent': colorType === EColorTypes.Accent },
99:             { 'button--size--fixed': fixedSize },
100:             shape !== EButtonShapes.Default && 'button--shape--' + {
```

### `src/components/Header/index.tsx:45`
```text
37:   user: userSelectors.user(state),
38:   language: configSelectors.language(state),
39:   activeOrders: ordersSelectors.activeOrders(state),
40:   selectedOrder: clientOrderSelectors.selectedOrder(state),
41: })
42: 
43: const mapDispatchToProps = {
44:   setLanguage: configActionCreators.setLanguage,
45:   setLoginModal: modalsActionCreators.setLoginModal,
46:   setProfileModal: modalsActionCreators.setProfileModal,
47: }
48: 
49: const connector = connect(mapStateToProps, mapDispatchToProps)
50: 
51: interface IProps extends ConnectedProps<typeof connector> {
52:   className?: string
53: }
54: 
55: function Header({
56:   user,
57:   language,
58:   activeOrders,
59:   selectedOrder,
```

### `src/components/Header/index.tsx:61`
```text
53: }
54: 
55: function Header({
56:   user,
57:   language,
58:   activeOrders,
59:   selectedOrder,
60:   setLanguage,
61:   setLoginModal,
62:   setProfileModal,
63:   className,
64: }: IProps) {
65:   const [languagesOpened, setLanguagesOpened] = useState(false)
66:   const [seconds, setSeconds] = useState(0)
67:   const [menuOpened, setMenuOpened] = useState(false)
68:   if (!menuOpened && languagesOpened)
69:     setLanguagesOpened(false)
70: 
71:   const location = useLocation()
72:   const navigate = useNavigate()
73: 
74:   const clientOrder = activeOrders?.find(item => item.b_id === selectedOrder)
75:   const driver = clientOrder?.drivers?.find(item =>
```

### `src/components/Header/index.tsx:201`
```text
193:         </button>}
194:       </div>
195:       <div className='header-logo'><img src={images.logo} alt="" /></div>
196:       <div className='header-avatar-wrapper'>
197:         <span className='header-user-name'>{user?.u_city ? `${( (window as any).data.cities[user?.u_city][ language.iso ??  (window as any).data.langs[(window as any).default_lang].iso ])},` : ''}</span>
198:         <span className='header-user-lng'>{language.iso.toUpperCase()}</span>
199:         <div
200:           className="avatar"
201:           onClick={e => setLoginModal(true)}
202:           style={{
203:             backgroundSize: avatarSize,
204:             backgroundImage: `url(${avatar})`,
205:           }}
206:         />
207: 
208:       </div>
209:     </header>
210:   )
211: }
212: 
213: export default connector(Header)
```

### `src/components/Input/index.tsx:38`
```text
30:   Textarea,
31:   Select,
32:   MaskedPhone,
33:   File,
34: }
35: 
36: export enum EInputStyles {
37:   Default,
38:   Login,
39:   RedDesign,
40: }
41: 
42: interface ISideCheckbox {
43:   value: boolean
44:   onClick: () => any
45:   component: React.ReactNode
46: }
47: 
48: interface IProps {
49:   inputType?: EInputTypes
50:   style?: EInputStyles,
51:   error?: string | null
52:   label?: string
```

### `src/components/Input/index.tsx:346`
```text
338:       cn(
339:         'input__field-wrapper',
340:         fieldWrapperClassName,
341:         {
342:           'input__field-wrapper--oneline': oneline || isDefaultValueUsed,
343:           'input__field-wrapper--margin-disabled': hideInput,
344:         },
345:         style !== EInputStyles.Default && 'input__field-wrapper--style--' + {
346:           [EInputStyles.Login]: 'login',
347:           [EInputStyles.RedDesign]: 'red-design',
348:         }[style],
349:       )
350:     }
351:     >
352:       <Helmet>
353:         <style>
354:           {`
355:             .input__label {
356:               color: ${SITE_CONSTANTS.PALETTE.primary.dark}
357:             }
358: 
359:             input[type="date"]::-webkit-calendar-picker-indicator {
360:               background-image: url("${images.date}");
```

### `src/components/Input/index.tsx:446`
```text
438:                       (
439:                         // eslint-disable-next-line jsx-a11y/alt-text
440:                         <img key={index} {...item as React.ComponentProps<'img'>} />
441:                       ) :
442:                       (
443:                         <Button
444:                           fixedSize={false}
445:                           shape={
446:                             style === EInputStyles.Login ?
447:                               EButtonShapes.Flat :
448:                               undefined
449:                           }
450:                           key={index}
451:                           {...item as React.ComponentProps<typeof Button>}
452:                         />
453:                       )
454:                   ))}
455:                 </div>}
456:                 <div
457:                   className={cn('input__suggestions', { 'input__suggestions--active': isFocused && suggestions?.length })}
458:                 >
459:                   {suggestions && suggestions.map((item, index) => (
460:                     <div
```

### `src/components/modals/CardModal.tsx:78`
```text
70:   cancelOrder: ordersActionCreators.cancel,
71:   getOrderStart: ordersDetailsActionCreators.getOrderStart,
72:   getOrderDestination: ordersDetailsActionCreators.getOrderDestination,
73:   setSelectedOrderId: orderActionCreators.setSelectedOrderId,
74:   setModal: modalsActionCreators.setOrderCardModal,
75:   setCancelDriverOrderModal: modalsActionCreators.setDriverCancelModal,
76:   setRatingModal: modalsActionCreators.setRatingModal,
77:   setAlarmModal: modalsActionCreators.setAlarmModal,
78:   setLoginModal: modalsActionCreators.setLoginModal,
79:   setMapModal: modalsActionCreators.setMapModal,
80:   setMessageModal: modalsActionCreators.setMessageModal,
81:   setActiveChat: modalsActionCreators.setActiveChat,
82: }
83: 
84: const connector = connect(mapStateToProps, mapDispatchToProps)
85: 
86: interface IFormValues {
87:   votingNumber: number
88:   performers_price: number
89: }
90: 
91: interface IProps extends ConnectedProps<typeof connector> {}
92: 
```

### `src/components/modals/CommentsModal/index.tsx:139`
```text
131:             ...register('custom'),
132:             placeholder: t(TRANSLATION.CUSTOM_COMMENT),
133:           }}
134:           style={EInputStyles.RedDesign}
135:         />
136:         <Button
137:           type="submit"
138:           buttonStyle={EButtonStyles.RedDesign}
139:           checkLogin={false}
140:           text={t(TRANSLATION.OK)}
141:         />
142:         {/* TODO voice recognition */}
143:         {/* TODO add icon */}
144:       </form>
145:     </Modal>
146:   )
147: }
148: 
149: export default connector(CommentsModal)
```

### `src/components/modals/ModalStack.tsx:17`
```text
9: import CommentsModal from './CommentsModal'
10: import DriverModal from './DriverModal'
11: import RatingModal from './RatingModal'
12: import OnTheWayModal from './OnTheWayModal'
13: import TieCardModal from './TieCardModal'
14: import CardDetailsModal from './CardDetailsModal'
15: import VoteModal from './VoteModal'
16: import PlaceModal from './SeatsModal'
17: import LoginModal from './LoginModal'
18: import AlarmModal from './AlarmModal'
19: import MapModal from './MapModal'
20: import TakePassengerModal from './TakePassengerModal'
21: import CancelDriverOrderModal from './DriverCancelModal'
22: import ProfileModal from './ProfileModal'
23: import CandidatesModal from './CandidatesModal'
24: import MessageModal from './MessageModal'
25: import WACodeModal from './LoginModal/WACodeModal'
26: import RefCodeModal from './LoginModal/RefCodeModal'
27: import CardModal from './CardModal'
28: 
29: const COMPONENTS = [
30:   [Chat, modalsSelectors.activeChat],
31:   [CancelOrderModal, modalsSelectors.isCancelModalOpen],
```

### `src/components/modals/ModalStack.tsx:25`
```text
17: import LoginModal from './LoginModal'
18: import AlarmModal from './AlarmModal'
19: import MapModal from './MapModal'
20: import TakePassengerModal from './TakePassengerModal'
21: import CancelDriverOrderModal from './DriverCancelModal'
22: import ProfileModal from './ProfileModal'
23: import CandidatesModal from './CandidatesModal'
24: import MessageModal from './MessageModal'
25: import WACodeModal from './LoginModal/WACodeModal'
26: import RefCodeModal from './LoginModal/RefCodeModal'
27: import CardModal from './CardModal'
28: 
29: const COMPONENTS = [
30:   [Chat, modalsSelectors.activeChat],
31:   [CancelOrderModal, modalsSelectors.isCancelModalOpen],
32:   [TimerModal, modalsSelectors.isPickTimeModalOpen],
33:   [CommentsModal, modalsSelectors.isCommentsModalOpen],
34:   [DriverModal, modalsSelectors.isDriverModalOpen],
35:   [RatingModal, modalsSelectors.isRatingModalOpen],
36:   [OnTheWayModal, modalsSelectors.isOnTheWayModalOpen],
37:   [TieCardModal, modalsSelectors.isTieCardModalOpen],
38:   [CardDetailsModal, modalsSelectors.isCardDetailsModalOpen],
39:   [VoteModal, modalsSelectors.isVoteModalOpen],
```

### `src/components/modals/ModalStack.tsx:26`
```text
18: import AlarmModal from './AlarmModal'
19: import MapModal from './MapModal'
20: import TakePassengerModal from './TakePassengerModal'
21: import CancelDriverOrderModal from './DriverCancelModal'
22: import ProfileModal from './ProfileModal'
23: import CandidatesModal from './CandidatesModal'
24: import MessageModal from './MessageModal'
25: import WACodeModal from './LoginModal/WACodeModal'
26: import RefCodeModal from './LoginModal/RefCodeModal'
27: import CardModal from './CardModal'
28: 
29: const COMPONENTS = [
30:   [Chat, modalsSelectors.activeChat],
31:   [CancelOrderModal, modalsSelectors.isCancelModalOpen],
32:   [TimerModal, modalsSelectors.isPickTimeModalOpen],
33:   [CommentsModal, modalsSelectors.isCommentsModalOpen],
34:   [DriverModal, modalsSelectors.isDriverModalOpen],
35:   [RatingModal, modalsSelectors.isRatingModalOpen],
36:   [OnTheWayModal, modalsSelectors.isOnTheWayModalOpen],
37:   [TieCardModal, modalsSelectors.isTieCardModalOpen],
38:   [CardDetailsModal, modalsSelectors.isCardDetailsModalOpen],
39:   [VoteModal, modalsSelectors.isVoteModalOpen],
40:   [PlaceModal, modalsSelectors.isSeatsModalOpen],
```

### `src/components/modals/ModalStack.tsx:41`
```text
33:   [CommentsModal, modalsSelectors.isCommentsModalOpen],
34:   [DriverModal, modalsSelectors.isDriverModalOpen],
35:   [RatingModal, modalsSelectors.isRatingModalOpen],
36:   [OnTheWayModal, modalsSelectors.isOnTheWayModalOpen],
37:   [TieCardModal, modalsSelectors.isTieCardModalOpen],
38:   [CardDetailsModal, modalsSelectors.isCardDetailsModalOpen],
39:   [VoteModal, modalsSelectors.isVoteModalOpen],
40:   [PlaceModal, modalsSelectors.isSeatsModalOpen],
41:   [LoginModal, modalsSelectors.isLoginModalOpen],
42:   [AlarmModal, modalsSelectors.isAlarmModalOpen],
43:   [MapModal, modalsSelectors.isMapModalOpen],
44:   [TakePassengerModal, modalsSelectors.isTakePassengerModalOpen],
45:   [CancelDriverOrderModal, modalsSelectors.isDriverCancelModalOpen],
46:   [ProfileModal, modalsSelectors.isProfileModalOpen],
47:   [CandidatesModal, modalsSelectors.isCandidatesModalOpen],
48:   [MessageModal, modalsSelectors.isMessageModalOpen],
49:   [WACodeModal, modalsSelectors.isWACodeModalOpen],
50:   [RefCodeModal, modalsSelectors.isRefCodeModalOpen],
51:   [CardModal, modalsSelectors.isOrderCardModalOpen],
52: ] as const
53: 
54: const modalsSelector = createSelector(
55:   (state: IRootState) => state,
```

### `src/components/modals/Overlay.tsx:20`
```text
12: 
13: const Overlay: React.FC<IProps> = ({ isOpen, onClick, children }) => (
14:   <div
15:     className={cn('overlay__wrapper', { 'overlay__wrapper--active': isOpen })}
16:   >
17:     <Helmet>
18:       <style>
19:         {`
20:         .modal form fieldset, .login-modal fieldset {
21:           border: 2px solid ${SITE_CONSTANTS.PALETTE.primary.main};
22:         }
23:         .modal form fieldset legend, .login-modal fieldset legend {
24:           color: ${SITE_CONSTANTS.PALETTE.primary.dark};
25:         }
26:         `}
27:       </style>
28:     </Helmet>
29:     <div
30:       className="overlay"
31:       onClick={onClick}
32:     />
33: 
34:     {children}
```

### `src/components/modals/Overlay.tsx:23`
```text
15:     className={cn('overlay__wrapper', { 'overlay__wrapper--active': isOpen })}
16:   >
17:     <Helmet>
18:       <style>
19:         {`
20:         .modal form fieldset, .login-modal fieldset {
21:           border: 2px solid ${SITE_CONSTANTS.PALETTE.primary.main};
22:         }
23:         .modal form fieldset legend, .login-modal fieldset legend {
24:           color: ${SITE_CONSTANTS.PALETTE.primary.dark};
25:         }
26:         `}
27:       </style>
28:     </Helmet>
29:     <div
30:       className="overlay"
31:       onClick={onClick}
32:     />
33: 
34:     {children}
35:   </div>
36: )
37: 
```

### `src/components/modals/ProfileModal.tsx:92`
```text
84:   'registration_plate',
85:   'color',
86:   'photo',
87:   'details',
88:   'cc_id',
89: ])
90: 
91: const mapStateToProps = (state: IRootState) => ({
92:   tokens: userSelectors.tokens(state),
93:   user: userSelectors.user(state),
94:   car: carsSelectors.userPrimaryCar(state),
95:   language: configSelectors.language(state),
96:   isOpen: modalsSelectors.isProfileModalOpen(state),
97: })
98: 
99: const mapDispatchToProps = {
100:   setProfileModal: modalsActionCreators.setProfileModal,
101:   setMessageModal: modalsActionCreators.setMessageModal,
102:   updateUser: userActionCreators.initUser,
103:   getUserCars: carsActionCreators.getUserCars,
104:   editCar: carsActionCreators.edit,
105:   setLanguage: configActionCreators.setLanguage,
106: }
```

### `src/components/modals/ProfileModal.tsx:113`
```text
105:   setLanguage: configActionCreators.setLanguage,
106: }
107: 
108: const connector = connect(mapStateToProps, mapDispatchToProps)
109: 
110: interface IProps extends ConnectedProps<typeof connector> {}
111: 
112: function ProfileModal({
113:   tokens,
114:   user,
115:   car,
116:   language,
117:   isOpen,
118:   setProfileModal,
119:   setMessageModal,
120:   updateUser,
121:   getUserCars,
122:   editCar,
123:   setLanguage,
124: }: IProps) {
125:   const onChangeAvatar = useCallback((e: any) => {
126:     const file = e.target.files[0]
127:     if (!user || !tokens || !file) return
```

### `src/components/modals/ProfileModal.tsx:127`
```text
119:   setMessageModal,
120:   updateUser,
121:   getUserCars,
122:   editCar,
123:   setLanguage,
124: }: IProps) {
125:   const onChangeAvatar = useCallback((e: any) => {
126:     const file = e.target.files[0]
127:     if (!user || !tokens || !file) return
128:     getBase64(file)
129:       .then((base64: any) => API.editUser({ u_photo: base64 }))
130:       .then(() => updateUser())
131:       .catch(error => alert(JSON.stringify(error)))
132:   }, [user, tokens])
133: 
134:   useEffect(() => {
135:     getUserCars()
136:   }, [])
137: 
138:   const [passportPhoto, setPassportPhoto] =
139:     useState<[number, File][] | null>(null)
140:   const [driverLicensePhoto, setDriverLicensePhoto] =
141:     useState<[number, File][] | null>(null)
```

### `src/components/modals/ProfileModal.tsx:132`
```text
124: }: IProps) {
125:   const onChangeAvatar = useCallback((e: any) => {
126:     const file = e.target.files[0]
127:     if (!user || !tokens || !file) return
128:     getBase64(file)
129:       .then((base64: any) => API.editUser({ u_photo: base64 }))
130:       .then(() => updateUser())
131:       .catch(error => alert(JSON.stringify(error)))
132:   }, [user, tokens])
133: 
134:   useEffect(() => {
135:     getUserCars()
136:   }, [])
137: 
138:   const [passportPhoto, setPassportPhoto] =
139:     useState<[number, File][] | null>(null)
140:   const [driverLicensePhoto, setDriverLicensePhoto] =
141:     useState<[number, File][] | null>(null)
142:   useEffect(() => {
143:     if (!isOpen) return
144:     const passportImgs = user?.u_details?.passport_photo || []
145:     const driverLicenseImgs = user?.u_details?.driver_license_photo || []
146:     Promise.all(passportImgs.map(getImageFile)).then(setPassportPhoto)
```

### `src/components/modals/ProfileModal.tsx:277`
```text
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
```

### `src/components/modals/ProfileModal.tsx:278`
```text
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
```

### `src/components/modals/TakePassengerModal.tsx:84`
```text
76:         fromLongitude: from?.longitude?.toString(),
77:         toAddress: to?.address,
78:         toLatitude: to?.latitude?.toString(),
79:         toLongitude: to?.longitude?.toString(),
80:       },
81:       seats,
82:     )
83:       .then(() => setTakePassengerModal({ isOpen: false }))
84:       .then(API.getAuthorizedUser)
85:       .then(setUser)
86:       .catch(error => {
87:         console.error(error)
88:         setMessageModal({ isOpen: true, message: t(TRANSLATION.ERROR), status: EStatuses.Fail })
89:       })
90:   }
91: 
92:   useEffect(() => {
93:     navigator.geolocation.getCurrentPosition(
94:       ({ coords }) => updateTakePassengerModal({ from: { latitude: coords.latitude, longitude: coords.longitude } }),
95:       error => console.error(error),
96:       { enableHighAccuracy: true },
97:     )
98:   }, [])
```

### `src/localization/common.ts:269`
```text
261:   LESS_THAN_600: 'less_than_600',
262:   LESS_TIME_ERROR: 'less_time_error',
263:   LIGHT: 'light',
264:   LIGHT_AUTO: 'light_auto',
265:   LIVING_ROOM: 'living_room',
266:   LOADING: 'loading',
267:   LOADING_P: 'loading_p',
268:   LOADING_PLACE_P: 'loading_place_p',
269:   LOGIN: 'login',
270:   LOGIN_FAIL: 'login_fail',
271:   LOGIN_SUCCESS: 'login_success',
272:   LOGOUT: 'logout',
273:   LOGOUT_FAIL: 'logout_fail',
274:   LOGOUT_SUCCESS: 'logout_success',
275:   LOVESEAT: 'loveseat',
276:   LOVE_SEAT: 'love_seat',
277:   LOW_CABINET: 'low_cabinet',
278:   L_BOX: 'l_box',
279:   MAP: 'map',
280:   MAP_FROM_NOT_SPECIFIED_ERROR: 'map_from_not_specified_error',
281:   MAP_TO_NOT_SPECIFIED_ERROR: 'map_to_not_specified_error',
282:   MARCH: 'march',
283:   MAY: 'may',
```

### `src/localization/common.ts:270`
```text
262:   LESS_TIME_ERROR: 'less_time_error',
263:   LIGHT: 'light',
264:   LIGHT_AUTO: 'light_auto',
265:   LIVING_ROOM: 'living_room',
266:   LOADING: 'loading',
267:   LOADING_P: 'loading_p',
268:   LOADING_PLACE_P: 'loading_place_p',
269:   LOGIN: 'login',
270:   LOGIN_FAIL: 'login_fail',
271:   LOGIN_SUCCESS: 'login_success',
272:   LOGOUT: 'logout',
273:   LOGOUT_FAIL: 'logout_fail',
274:   LOGOUT_SUCCESS: 'logout_success',
275:   LOVESEAT: 'loveseat',
276:   LOVE_SEAT: 'love_seat',
277:   LOW_CABINET: 'low_cabinet',
278:   L_BOX: 'l_box',
279:   MAP: 'map',
280:   MAP_FROM_NOT_SPECIFIED_ERROR: 'map_from_not_specified_error',
281:   MAP_TO_NOT_SPECIFIED_ERROR: 'map_to_not_specified_error',
282:   MARCH: 'march',
283:   MAY: 'may',
284:   MEDIUM_BOX: 'medium_box',
```

### `src/localization/common.ts:271`
```text
263:   LIGHT: 'light',
264:   LIGHT_AUTO: 'light_auto',
265:   LIVING_ROOM: 'living_room',
266:   LOADING: 'loading',
267:   LOADING_P: 'loading_p',
268:   LOADING_PLACE_P: 'loading_place_p',
269:   LOGIN: 'login',
270:   LOGIN_FAIL: 'login_fail',
271:   LOGIN_SUCCESS: 'login_success',
272:   LOGOUT: 'logout',
273:   LOGOUT_FAIL: 'logout_fail',
274:   LOGOUT_SUCCESS: 'logout_success',
275:   LOVESEAT: 'loveseat',
276:   LOVE_SEAT: 'love_seat',
277:   LOW_CABINET: 'low_cabinet',
278:   L_BOX: 'l_box',
279:   MAP: 'map',
280:   MAP_FROM_NOT_SPECIFIED_ERROR: 'map_from_not_specified_error',
281:   MAP_TO_NOT_SPECIFIED_ERROR: 'map_to_not_specified_error',
282:   MARCH: 'march',
283:   MAY: 'may',
284:   MEDIUM_BOX: 'medium_box',
285:   MENU: 'menu',
```

### `src/localization/common.ts:414`
```text
406:   SUCCESS_REGISTER_MESSAGE: 'success_register_message',
407:   SEND_PARCEL: 'send_parcel',
408:   SEPTEMBER: 'september',
409:   SET_ADJUSTABLE_POINT: 'set_adjustable_point',
410:   SET_FROM_POINT: 'set_from_point',
411:   SET_TO_POINT: 'set_to_point',
412:   SHELF: 'shelf',
413:   SHIPPING_APPLICATION_P: 'shipping_application_p',
414:   SIGNIN: 'signin',
415:   SIGNUP: 'signup',
416:   SIGN_IN: 'sign_in',
417:   SIGN_IN_HEADER: 'sign_in_header',
418:   SMALL_BOX: 'small_box',
419:   SMALL_FRIDGE: 'small_fridge',
420:   SOFA: 'sofa',
421:   SOFA2: 'sofa2',
422:   SOFA_BED: 'sofa_bed',
423:   SOFA_SECTIONAL: 'sofa_sectional',
424:   STARS: 'stars',
425:   START_POINT: 'start_point',
426:   TIME_P: 'time_p',
427:   START_TIME: 'start_time',
428:   STATE: 'state',
```

### `src/localization/common.ts:450`
```text
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
```

### `src/localization/common.ts:482`
```text
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
```

### `src/localization/common.ts:483`
```text
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
```

### `src/pages/Driver/Orders.tsx:60`
```text
52:   const [statusID, setStatusID] = useState(statuses[0].id)
53: 
54:   const navigate = useNavigate()
55: 
56:   const handleOrderClick = (id: string) => navigate(`/driver-order/${id}`)
57: 
58:   const handleDrovePassengerClick = () => {
59:     API.setOutDrive(true)
60:       .then(API.getAuthorizedUser)
61:       .then((user) => setUser(user))
62:   }
63: 
64:   const candidateOrders = activeOrders?.filter(item => {
65:     return (
66:       item.drivers?.length &&
67:       item.drivers.find(i => i.u_id === user?.u_id && i.c_state === EBookingDriverState.Considering)
68:     )
69:   })
70: 
71:   const activeOrdersWithoutCandidates = activeOrders?.filter(item => !candidateOrders?.includes(item))
72: 
73:   return (
74:     <PageSection className="driver">
```

### `src/pages/Driver/index.tsx:30`
```text
22:   historyOrders: ordersSelectors.historyOrders(state),
23:   user: userSelectors.user(state),
24: })
25: 
26: const mapDispatchToProps = {
27:   watchActiveOrders: ordersActionCreators.watchActiveOrders,
28:   watchReadyOrders: ordersActionCreators.watchReadyOrders,
29:   watchHistoryOrders: ordersActionCreators.watchHistoryOrders,
30:   setLoginModal: modalsActionCreators.setLoginModal,
31: }
32: 
33: const connector = connect(mapStateToProps, mapDispatchToProps)
34: 
35: export const OrderAddressContext = createContext<{ ordersAddressRef: React.RefObject<{
36:   [orderId: string]: IAddressPoint;
37: }> }|null>(null)
38: 
39: export enum EDriverTabs {
40:   Map = 'map',
41:   Lite = 'lite',
42:   Detailed = 'detailed'
43: }
44: 
```

### `src/pages/Driver/index.tsx:55`
```text
47: const Driver: React.FC<IProps> = ({
48:   activeOrders,
49:   readyOrders,
50:   historyOrders,
51:   user,
52:   watchActiveOrders,
53:   watchHistoryOrders,
54:   watchReadyOrders,
55:   setLoginModal,
56: }) => {
57: 
58:   const { tab = EDriverTabs.Lite } = useQuery()
59: 
60:   const navigate = useNavigate()
61: 
62:   const ordersAddressRef = useRef<{ [orderId:string]: IAddressPoint }>({})
63: 
64:   useEffect(watchActiveOrders, [])
65:   useEffect(watchReadyOrders, [])
66:   useEffect(watchHistoryOrders, [])
67: 
68:   if (user?.u_role !== EUserRoles.Driver) {
69:     return (
```

### `src/pages/Driver/index.tsx:72`
```text
64:   useEffect(watchActiveOrders, [])
65:   useEffect(watchReadyOrders, [])
66:   useEffect(watchHistoryOrders, [])
67: 
68:   if (user?.u_role !== EUserRoles.Driver) {
69:     return (
70:       <ErrorFrame
71:         renderImage={() => (
72:           <div className="errorIcon" onClick={() => setLoginModal(true)}>
73:             <img src={images.avatar} alt={t(TRANSLATION.ERROR)} style={{ marginTop: '50px' }}/>
74:           </div>
75:         )}
76:         title={t(TRANSLATION.UNAUTHORIZED_ACCESS)}
77:       />
78:     )
79:   }
80: 
81:   return (
82:     <>
83:       <div className="driver-tabs">
84:         <button
85:           onClick={() => navigate(`?tab=${EDriverTabs.Lite}`)}
86:           className={cn('driver-tabs__tab', { 'driver-tabs__tab--active': tab === EDriverTabs.Lite })}
```

### `src/pages/Driver/index.tsx:76`
```text
68:   if (user?.u_role !== EUserRoles.Driver) {
69:     return (
70:       <ErrorFrame
71:         renderImage={() => (
72:           <div className="errorIcon" onClick={() => setLoginModal(true)}>
73:             <img src={images.avatar} alt={t(TRANSLATION.ERROR)} style={{ marginTop: '50px' }}/>
74:           </div>
75:         )}
76:         title={t(TRANSLATION.UNAUTHORIZED_ACCESS)}
77:       />
78:     )
79:   }
80: 
81:   return (
82:     <>
83:       <div className="driver-tabs">
84:         <button
85:           onClick={() => navigate(`?tab=${EDriverTabs.Lite}`)}
86:           className={cn('driver-tabs__tab', { 'driver-tabs__tab--active': tab === EDriverTabs.Lite })}
87:         >
88:           {t(TRANSLATION.LIGHT)}
89:         </button>
90:         <button
```

### `src/pages/Order/index.tsx:42`
```text
34: })
35: 
36: const mapDispatchToProps = {
37:   getOrder: orderActionCreators.getOrder,
38:   setOrder: orderActionCreators.setOrder,
39:   setCancelDriverOrderModal: modalsActionCreators.setDriverCancelModal,
40:   setRatingModal: modalsActionCreators.setRatingModal,
41:   setAlarmModal: modalsActionCreators.setAlarmModal,
42:   setLoginModal: modalsActionCreators.setLoginModal,
43:   setMapModal: modalsActionCreators.setMapModal,
44:   setMessageModal: modalsActionCreators.setMessageModal,
45: }
46: 
47: const connector = connect(mapStateToProps, mapDispatchToProps)
48: 
49: interface IFormValues {
50:   votingNumber: number
51:   performers_price: number
52: }
53: 
54: interface IProps extends ConnectedProps<typeof connector> {}
55: 
56: const Order: React.FC<IProps> = ({
```

### `src/pages/Passenger/VotingForm.tsx:44`
```text
36:   time: clientOrderSelectors.time(state),
37:   phone: clientOrderSelectors.phone(state),
38:   user: userSelectors.user(state),
39: })
40: 
41: const mapDispatchToProps = {
42:   setPickTimeModal: modalsActionCreators.setPickTimeModal,
43:   setCommentsModal: modalsActionCreators.setCommentsModal,
44:   setLoginModal: modalsActionCreators.setLoginModal,
45:   watchActiveOrders: ordersActionCreators.watchActiveOrders,
46:   createOrder: ordersActionCreators.create,
47:   setPhone: clientOrderActionCreators.setPhone,
48:   resetClientOrder: clientOrderActionCreators.reset,
49: }
50: 
51: const connector = connect(mapStateToProps, mapDispatchToProps)
52: 
53: interface IProps extends ConnectedProps<typeof connector> {
54:   isExpanded: boolean
55:   setIsExpanded: React.Dispatch<React.SetStateAction<boolean>>
56:   syncFrom: () => void
57:   syncTo: () => void
58:   onSubmit: (data: Awaited<ReturnType<typeof API.postDrive>>) => void
```

### `src/pages/Passenger/VotingForm.tsx:73`
```text
65:   from,
66:   to,
67:   comments,
68:   time,
69:   phone,
70:   user,
71:   setPickTimeModal,
72:   setCommentsModal,
73:   setLoginModal,
74:   watchActiveOrders,
75:   createOrder,
76:   setPhone,
77:   resetClientOrder,
78:   isExpanded,
79:   setIsExpanded,
80:   syncFrom,
81:   syncTo,
82:   onSubmit,
83:   minimizedPartRef,
84:   noSwipeElementsRef,
85: }: IProps) {
86: 
87:   const carSliderRef = useRef<HTMLDivElement>(null)
```

### `src/pages/Passenger/VotingForm.tsx:135`
```text
127:       setPhoneError(phoneError)
128:       setIsExpanded(true)
129:       error = true
130:     }
131:     if (error)
132:       return
133: 
134:     if (!user) {
135:       setLoginModal(true)
136:       return
137:     }
138: 
139:     const commentObj: any = {}
140:     commentObj['b_comments'] = comments.ids || []
141:     comments.custom &&
142:       (commentObj['b_custom_comment'] = comments.custom)
143:     comments.flightNumber &&
144:       (commentObj['b_flight_number'] = comments.flightNumber)
145:     comments.placard && (commentObj['b_placard'] = comments.placard)
146: 
147:     const startTime = moment(voting || time === 'now' ? undefined : time)
148: 
149:     setSubmitting(true)
```

### `src/pages/Passenger/VotingForm.tsx:185`
```text
177:         (error as any)?.message?.toString() ||
178:         t(TRANSLATION.ERROR),
179:       )
180:       console.error(error)
181:     }
182:     setSubmitting(false)
183:   }, [
184:     from, to, comments, time, phone, user,
185:     store, setLoginModal, createOrder,
186:     setIsExpanded, onSubmit,
187:   ])
188: 
189:   const [submitting, setSubmitting] = useState(false)
190:   const [submitError, setSubmitError] = useState<string | null>(null)
191: 
192:   const submitButtons = (
193:     <div className="passenger-voting-form__order-button-wrapper">
194:       {useMemo(() =>
195:         <>
196:           <Button
197:             wrapperProps={{ className: 'passenger-voting-form__order-button' }}
198:             buttonStyle={EButtonStyles.RedDesign}
199:             type="submit"
```

### `src/pages/Passenger/VotingForm.tsx:200`
```text
192:   const submitButtons = (
193:     <div className="passenger-voting-form__order-button-wrapper">
194:       {useMemo(() =>
195:         <>
196:           <Button
197:             wrapperProps={{ className: 'passenger-voting-form__order-button' }}
198:             buttonStyle={EButtonStyles.RedDesign}
199:             type="submit"
200:             checkLogin={false}
201:             text={t(TRANSLATION.VOTE, { toUpper: false })}
202:             onClick={() => submit(true)}
203:             disabled={!available || submitting}
204:           />
205:           <Button
206:             wrapperProps={{ className: 'passenger-voting-form__order-button' }}
207:             buttonStyle={EButtonStyles.RedDesign}
208:             type="submit"
209:             checkLogin={false}
210:             text={t(TRANSLATION.TO_ORDER, { toUpper: false })}
211:             onClick={() => submit()}
212:             disabled={!available || submitting}
213:           />
214:         </>
```

### `src/pages/Passenger/VotingForm.tsx:209`
```text
201:             text={t(TRANSLATION.VOTE, { toUpper: false })}
202:             onClick={() => submit(true)}
203:             disabled={!available || submitting}
204:           />
205:           <Button
206:             wrapperProps={{ className: 'passenger-voting-form__order-button' }}
207:             buttonStyle={EButtonStyles.RedDesign}
208:             type="submit"
209:             checkLogin={false}
210:             text={t(TRANSLATION.TO_ORDER, { toUpper: false })}
211:             onClick={() => submit()}
212:             disabled={!available || submitting}
213:           />
214:         </>
215:       , [available, submitting, submit])}
216:       {submitError &&
217:         <span className="passenger-voting-form__order-button-error">
218:           {submitError}
219:         </span>
220:       }
221:     </div>
222:   )
223: 
```

### `src/state/modals/actionCreators.ts:31`
```text
23:   return { type: ActionTypes.SET_CARD_DETAILS_MODAL, payload }
24: }
25: export const setVoteModal = (payload: IModalsState['isVoteModalOpen']): TAction => {
26:   return { type: ActionTypes.SET_VOTE_MODAL, payload }
27: }
28: export const setSeatsModal = (payload: IModalsState['isSeatsModalOpen']): TAction => {
29:   return { type: ActionTypes.SET_SEATS_MODAL, payload }
30: }
31: export const setLoginModal = (payload: IModalsState['isLoginModalOpen']): TAction => {
32:   return { type: ActionTypes.SET_LOGIN_MODAL, payload }
33: }
34: export const setAlarmModal = (payload: Partial<IModalsState['alarmModal']>): TAction => {
35:   return { type: ActionTypes.SET_ALARM_MODAL, payload }
36: }
37: export const setWACodeModal = (payload: Partial<IModalsState['WACodeModal']>): TAction => {
38:   return { type: ActionTypes.SET_WACODE_MODAL, payload }
39: }
40: export const setRefCodeModal = (payload: Partial<IModalsState['RefCodeModal']>): TAction => {
41:   return { type: ActionTypes.SET_REFCODE_MODAL, payload }
42: }
43: export const setTakePassengerModal = (payload: IModalsState['takePassengerModal']): TAction => {
44:   return { type: ActionTypes.SET_TAKE_PASSENGER_MODAL, payload }
45: }
```

### `src/state/modals/actionCreators.ts:32`
```text
24: }
25: export const setVoteModal = (payload: IModalsState['isVoteModalOpen']): TAction => {
26:   return { type: ActionTypes.SET_VOTE_MODAL, payload }
27: }
28: export const setSeatsModal = (payload: IModalsState['isSeatsModalOpen']): TAction => {
29:   return { type: ActionTypes.SET_SEATS_MODAL, payload }
30: }
31: export const setLoginModal = (payload: IModalsState['isLoginModalOpen']): TAction => {
32:   return { type: ActionTypes.SET_LOGIN_MODAL, payload }
33: }
34: export const setAlarmModal = (payload: Partial<IModalsState['alarmModal']>): TAction => {
35:   return { type: ActionTypes.SET_ALARM_MODAL, payload }
36: }
37: export const setWACodeModal = (payload: Partial<IModalsState['WACodeModal']>): TAction => {
38:   return { type: ActionTypes.SET_WACODE_MODAL, payload }
39: }
40: export const setRefCodeModal = (payload: Partial<IModalsState['RefCodeModal']>): TAction => {
41:   return { type: ActionTypes.SET_REFCODE_MODAL, payload }
42: }
43: export const setTakePassengerModal = (payload: IModalsState['takePassengerModal']): TAction => {
44:   return { type: ActionTypes.SET_TAKE_PASSENGER_MODAL, payload }
45: }
46: export const updateTakePassengerModal = (payload: Partial<IModalsState['takePassengerModal']>): TAction => {
```

### `src/state/modals/constants.ts:19`
```text
11:   SET_PICK_TIME_MODAL: `${prefix}/SET_PICK_TIME_MODAL`,
12:   SET_COMMENTS_MODAL: `${prefix}/SET_COMMENTS_MODAL`,
13:   SET_DRIVER_MODAL: `${prefix}/SET_DRIVER_MODAL`,
14:   SET_RATING_MODAL: `${prefix}/SET_RATING_MODAL`,
15:   SET_TIE_CARD_MODAL: `${prefix}/SET_TIE_CARD_MODAL`,
16:   SET_CARD_DETAILS_MODAL: `${prefix}/SET_CARD_DETAILS_MODAL`,
17:   SET_VOTE_MODAL: `${prefix}/SET_VOTE_MODAL`,
18:   SET_SEATS_MODAL: `${prefix}/SET_PLACE_MODAL`,
19:   SET_LOGIN_MODAL: `${prefix}/SET_LOGIN_MODAL`,
20:   SET_ALARM_MODAL: `${prefix}/SET_ALARM_MODAL`,
21:   SET_WACODE_MODAL: `${prefix}/SET_WACODE_MODAL`,
22:   SET_REFCODE_MODAL: `${prefix}/SET_REFCODE_MODAL`,
23:   SET_TAKE_PASSENGER_MODAL: `${prefix}/SET_TAKE_PASSENGER_MODAL`,
24:   UPDATE_TAKE_PASSENGER_MODAL: `${prefix}/UPDATE_TAKE_PASSENGER_MODAL`,
25:   SET_TAKE_PASSENGER_MODAL_FROM_REQUEST: `${prefix}/SET_TAKE_PASSENGER_MODAL_FROM_REQUEST`,
26:   SET_TAKE_PASSENGER_MODAL_FROM: `${prefix}/SET_TAKE_PASSENGER_MODAL_FROM`,
27:   SET_TAKE_PASSENGER_MODAL_TO_REQUEST: `${prefix}/SET_TAKE_PASSENGER_MODAL_TO_REQUEST`,
28:   SET_TAKE_PASSENGER_MODAL_TO: `${prefix}/SET_TAKE_PASSENGER_MODAL_TO`,
29:   SET_DRIVER_CANCEL_MODAL: `${prefix}/SET_DRIVER_CANCEL_MODAL`,
30:   SET_ON_THE_WAY_MODAL: `${prefix}/SET_ON_THE_WAY_MODAL`,
31:   SET_MAP_MODAL: `${prefix}/SET_MAP_MODAL`,
32:   SET_MESSAGE_MODAL: `${prefix}/SET_MESSAGE_MODAL`,
33:   CLOSE_ALL_MODALS: `${prefix}/CLOSE_ALL_MODALS`,
```

### `src/state/modals/constants.ts:57`
```text
49:   isCancelModalOpen: boolean
50:   isPickTimeModalOpen: boolean
51:   isCommentsModalOpen: boolean
52:   isDriverModalOpen: boolean
53:   isTieCardModalOpen: boolean
54:   isCardDetailsModalOpen: boolean
55:   isVoteModalOpen: boolean
56:   isSeatsModalOpen: boolean
57:   isLoginModalOpen: boolean
58:   isCandidatesModalOpen: boolean
59:   isShowSwitchersMenu: boolean
60:   WACodeModal: {
61:     isOpen: boolean
62:     login: any
63:     data: any
64:   }
65:   RefCodeModal: {
66:     isOpen: boolean
67:     login: any
68:     data: any
69:   }
70:   alarmModal: {
71:     isOpen: boolean
```

### `src/state/modals/constants.ts:62`
```text
54:   isCardDetailsModalOpen: boolean
55:   isVoteModalOpen: boolean
56:   isSeatsModalOpen: boolean
57:   isLoginModalOpen: boolean
58:   isCandidatesModalOpen: boolean
59:   isShowSwitchersMenu: boolean
60:   WACodeModal: {
61:     isOpen: boolean
62:     login: any
63:     data: any
64:   }
65:   RefCodeModal: {
66:     isOpen: boolean
67:     login: any
68:     data: any
69:   }
70:   alarmModal: {
71:     isOpen: boolean
72:     seconds: number
73:   }
74:   takePassengerModal: {
75:     isOpen: boolean
76:     from?: IAddressPoint | null | undefined
```

### `src/state/modals/constants.ts:67`
```text
59:   isShowSwitchersMenu: boolean
60:   WACodeModal: {
61:     isOpen: boolean
62:     login: any
63:     data: any
64:   }
65:   RefCodeModal: {
66:     isOpen: boolean
67:     login: any
68:     data: any
69:   }
70:   alarmModal: {
71:     isOpen: boolean
72:     seconds: number
73:   }
74:   takePassengerModal: {
75:     isOpen: boolean
76:     from?: IAddressPoint | null | undefined
77:     to?: IAddressPoint | null | undefined
78:   }
79:   isDriverCancelModalOpen: boolean
80:   isOnTheWayModalOpen: boolean
81:   mapModal: {
```

### `src/state/modals/reducer.ts:25`
```text
17: }
18: export const defaultMapModal = {
19:   isOpen: false,
20:   type: EMapModalTypes.Client,
21:   defaultCenter: null,
22: }
23: export const defaultWACodeModal = {
24:   isOpen: false,
25:   login: null,
26:   data: null,
27: }
28: export const defaultRefCodeModal = {
29:   isOpen: false,
30:   login: null,
31:   data: null,
32: }
33: export const defaultTakePassengerModal = {
34:   isOpen: false,
35:   from: null,
36:   to: null,
37: }
38: export const defaultRatingModal = {
39:   isOpen: false,
```

### `src/state/modals/reducer.ts:30`
```text
22: }
23: export const defaultWACodeModal = {
24:   isOpen: false,
25:   login: null,
26:   data: null,
27: }
28: export const defaultRefCodeModal = {
29:   isOpen: false,
30:   login: null,
31:   data: null,
32: }
33: export const defaultTakePassengerModal = {
34:   isOpen: false,
35:   from: null,
36:   to: null,
37: }
38: export const defaultRatingModal = {
39:   isOpen: false,
40:   orderID: null,
41: }
42: export const defaultProfileModal = {
43:   isOpen: false,
44:   status: EStatuses.Default,
```

### `src/state/modals/reducer.ts:65`
```text
57:   isDriverModalOpen: false,
58:   isTieCardModalOpen: false,
59:   isCardDetailsModalOpen: false,
60:   isVoteModalOpen: false,
61:   isShowSwitchersMenu: false,
62:   WACodeModal: { ...defaultWACodeModal },
63:   RefCodeModal: { ...defaultRefCodeModal },
64:   isSeatsModalOpen: false,
65:   isLoginModalOpen: false,
66:   isDriverCancelModalOpen: false,
67:   isOnTheWayModalOpen: false,
68:   isCandidatesModalOpen: false,
69:   profileModal: { ...defaultProfileModal },
70:   alarmModal: { ...defaultAlarmModal },
71:   messageModal: { ...defaultMessageModal },
72:   mapModal: { ...defaultMapModal },
73:   takePassengerModal: { ...defaultTakePassengerModal },
74:   ratingModal: { ...defaultRatingModal },
75:   activeChat: null,
76:   deleteFilesModal: { ...defaultDeleteFilesModal },
77:   orderCardModal: defaultOrderCardModal,
78: })
79: 
```

### `src/state/modals/reducer.ts:111`
```text
103:       return state
104:         .set('isCardDetailsModalOpen', payload)
105:     case ActionTypes.SET_VOTE_MODAL:
106:       return state
107:         .set('isVoteModalOpen', payload)
108:     case ActionTypes.SET_SEATS_MODAL:
109:       return state
110:         .set('isSeatsModalOpen', payload)
111:     case ActionTypes.SET_LOGIN_MODAL:
112:       return state
113:         .set('isLoginModalOpen', payload)
114:     case ActionTypes.SET_CANDIDATES_MODAL:
115:       return state
116:         .set('isCandidatesModalOpen', payload)
117:     case ActionTypes.SET_WACODE_MODAL:
118:       return state
119:         .set('WACodeModal', payload)
120:     case ActionTypes.SET_REFCODE_MODAL:
121:       return state
122:         .set('RefCodeModal', payload)
123:     case ActionTypes.SET_ALARM_MODAL:
124:       return state
125:         .set('alarmModal', { ...payload, seconds: payload.seconds || (payload.isOpen ? ALARM_SECONDS : 0) })
```

### `src/state/modals/reducer.ts:113`
```text
105:     case ActionTypes.SET_VOTE_MODAL:
106:       return state
107:         .set('isVoteModalOpen', payload)
108:     case ActionTypes.SET_SEATS_MODAL:
109:       return state
110:         .set('isSeatsModalOpen', payload)
111:     case ActionTypes.SET_LOGIN_MODAL:
112:       return state
113:         .set('isLoginModalOpen', payload)
114:     case ActionTypes.SET_CANDIDATES_MODAL:
115:       return state
116:         .set('isCandidatesModalOpen', payload)
117:     case ActionTypes.SET_WACODE_MODAL:
118:       return state
119:         .set('WACodeModal', payload)
120:     case ActionTypes.SET_REFCODE_MODAL:
121:       return state
122:         .set('RefCodeModal', payload)
123:     case ActionTypes.SET_ALARM_MODAL:
124:       return state
125:         .set('alarmModal', { ...payload, seconds: payload.seconds || (payload.isOpen ? ALARM_SECONDS : 0) })
126:     case ActionTypes.SET_TAKE_PASSENGER_MODAL:
127:       return state
```

### `src/state/modals/reducer.ts:172`
```text
164:         .set('isCancelModalOpen', false)
165:         .set('isPickTimeModalOpen', false)
166:         .set('isCommentsModalOpen', false)
167:         .set('isDriverModalOpen', false)
168:         .set('isTieCardModalOpen', false)
169:         .set('isCardDetailsModalOpen', false)
170:         .set('isVoteModalOpen', false)
171:         .set('isSeatsModalOpen', false)
172:         .set('isLoginModalOpen', false)
173:         .set('isDriverCancelModalOpen', false)
174:         .set('isOnTheWayModalOpen', false)
175:         .set('isCandidatesModalOpen', false)
176:         .set('takePassengerModal', { ...defaultTakePassengerModal })
177:         .set('alarmModal', { ...defaultAlarmModal })
178:         .set('mapModal', { ...defaultMapModal })
179:         .set('messageModal', { ...defaultMessageModal })
180:         .set('ratingModal', { ...defaultRatingModal })
181:         .set('profileModal', { ...defaultProfileModal })
182:         .set('deleteFilesModal', { ...defaultDeleteFilesModal })
183:         .set('orderCardModal', defaultOrderCardModal)
184:     case ActionTypes.SET_SHOW_SWITCHERS_MENU:
185:       return state.set('isShowSwitchersMenu', payload)
186:     default:
```

### `src/state/modals/selectors.ts:18`
```text
10: export const isWACodeModalOpen = createSelector(moduleSelector, state => state.WACodeModal)
11: export const isRefCodeModalOpen = createSelector(moduleSelector, state => state.RefCodeModal)
12: export const isRatingModalOpen = createSelector(moduleSelector, state => state.ratingModal.isOpen)
13: export const ratingModalOrderID = createSelector(moduleSelector, state => state.ratingModal.orderID)
14: export const isTieCardModalOpen = createSelector(moduleSelector, state => state.isTieCardModalOpen)
15: export const isCardDetailsModalOpen = createSelector(moduleSelector, state => state.isCardDetailsModalOpen)
16: export const isVoteModalOpen = createSelector(moduleSelector, state => state.isVoteModalOpen)
17: export const isSeatsModalOpen = createSelector(moduleSelector, state => state.isSeatsModalOpen)
18: export const isLoginModalOpen = createSelector(moduleSelector, state => state.isLoginModalOpen)
19: export const isAlarmModalOpen = createSelector(moduleSelector, state => state.alarmModal.isOpen)
20: export const alarmModalSeconds = createSelector(moduleSelector, state => state.alarmModal.seconds)
21: export const isDriverCancelModalOpen = createSelector(moduleSelector, state => state.isDriverCancelModalOpen)
22: export const isOnTheWayModalOpen = createSelector(moduleSelector, state => state.isOnTheWayModalOpen)
23: export const isMapModalOpen = createSelector(moduleSelector, state => state.mapModal.isOpen)
24: export const mapModalType = createSelector(moduleSelector, state => state.mapModal.type)
25: export const mapModalDefaultCenter = createSelector(moduleSelector, state => state.mapModal.defaultCenter)
26: export const isMessageModalOpen = createSelector(moduleSelector, state => state.messageModal.isOpen)
27: export const messageModalMessage = createSelector(moduleSelector, state => state.messageModal.message)
28: export const messageModalStatus = createSelector(moduleSelector, state => state.messageModal.status)
29: export const isTakePassengerModalOpen = createSelector(moduleSelector, state => state.takePassengerModal.isOpen)
30: export const takePassengerModalFrom = createSelector(moduleSelector, state => state.takePassengerModal.from)
31: export const takePassengerModalTo = createSelector(moduleSelector, state => state.takePassengerModal.to)
32: export const activeChat = createSelector(moduleSelector, state => state.activeChat)
```

### `src/state/user/actionCreators.ts:11`
```text
3: import { ActionTypes, IUserState } from './constants'
4: 
5: export const register = (payload: Parameters<typeof API.register>[0] & {
6:   u_car?: Parameters<typeof createUserCar>[0]
7: }) => {
8:   return { type: ActionTypes.REGISTER_REQUEST, payload }
9: }
10: 
11: export const login = (payload: Parameters<typeof API.login>[0] & {
12:   navigate: (location: string) => void
13: }) => {
14:   return { type: ActionTypes.LOGIN_REQUEST, payload }
15: }
16: 
17: export const googleLogin = (payload: Parameters<typeof API.googleLogin>[0] & {
18:   navigate: (location: string) => void
19: }) => {
20:   return { type: ActionTypes.GOOGLE_LOGIN_REQUEST, payload }
21: }
22: 
23: export const logout = () => {
24:   return { type: ActionTypes.LOGOUT_REQUEST }
25: }
```

### `src/state/user/actionCreators.ts:14`
```text
6:   u_car?: Parameters<typeof createUserCar>[0]
7: }) => {
8:   return { type: ActionTypes.REGISTER_REQUEST, payload }
9: }
10: 
11: export const login = (payload: Parameters<typeof API.login>[0] & {
12:   navigate: (location: string) => void
13: }) => {
14:   return { type: ActionTypes.LOGIN_REQUEST, payload }
15: }
16: 
17: export const googleLogin = (payload: Parameters<typeof API.googleLogin>[0] & {
18:   navigate: (location: string) => void
19: }) => {
20:   return { type: ActionTypes.GOOGLE_LOGIN_REQUEST, payload }
21: }
22: 
23: export const logout = () => {
24:   return { type: ActionTypes.LOGOUT_REQUEST }
25: }
26: 
27: export const remindPassword = (payload: Parameters<typeof API.remindPassword>[0]) => {
28:   return { type: ActionTypes.REMIND_PASSWORD_REQUEST, payload }
```

### `src/state/user/actionCreators.ts:17`
```text
9: }
10: 
11: export const login = (payload: Parameters<typeof API.login>[0] & {
12:   navigate: (location: string) => void
13: }) => {
14:   return { type: ActionTypes.LOGIN_REQUEST, payload }
15: }
16: 
17: export const googleLogin = (payload: Parameters<typeof API.googleLogin>[0] & {
18:   navigate: (location: string) => void
19: }) => {
20:   return { type: ActionTypes.GOOGLE_LOGIN_REQUEST, payload }
21: }
22: 
23: export const logout = () => {
24:   return { type: ActionTypes.LOGOUT_REQUEST }
25: }
26: 
27: export const remindPassword = (payload: Parameters<typeof API.remindPassword>[0]) => {
28:   return { type: ActionTypes.REMIND_PASSWORD_REQUEST, payload }
29: }
30: 
31: // export const clearMessages = () => {
```

### `src/state/user/actionCreators.ts:20`
```text
12:   navigate: (location: string) => void
13: }) => {
14:   return { type: ActionTypes.LOGIN_REQUEST, payload }
15: }
16: 
17: export const googleLogin = (payload: Parameters<typeof API.googleLogin>[0] & {
18:   navigate: (location: string) => void
19: }) => {
20:   return { type: ActionTypes.GOOGLE_LOGIN_REQUEST, payload }
21: }
22: 
23: export const logout = () => {
24:   return { type: ActionTypes.LOGOUT_REQUEST }
25: }
26: 
27: export const remindPassword = (payload: Parameters<typeof API.remindPassword>[0]) => {
28:   return { type: ActionTypes.REMIND_PASSWORD_REQUEST, payload }
29: }
30: 
31: // export const clearMessages = () => {
32: //   return { type: ActionTypes.CLEAR_MESSAGES }
33: // }
34: 
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
17:   GOOGLE_LOGIN_START: `${prefix}/GOOGLE_LOGIN_START`,
18:   GOOGLE_LOGIN_SUCCESS: `${prefix}/GOOGLE_LOGIN_SUCCESS`,
```

### `src/state/user/constants.ts:16`
```text
8: const prefix = `${appName}/${moduleName}`
9: 
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
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
```

### `src/state/user/constants.ts:17`
```text
9: 
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
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
```

### `src/state/user/constants.ts:18`
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
29:   LOGOUT_SUCCESS: `${prefix}/LOGOUT_SUCCESS`,
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
```

### `src/state/user/constants.ts:19`
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
30:   LOGOUT_FAIL: `${prefix}/LOGOUT_FAIL`,
31: 
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
```

### `src/state/user/constants.ts:21`
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
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
```

### `src/state/user/constants.ts:22`
```text
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
32:   REMIND_PASSWORD_REQUEST: `${prefix}/REMIND_PASSWORD_REQUEST`,
33:   REMIND_PASSWORD_START: `${prefix}/REMIND_PASSWORD_START`,
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
```

### `src/state/user/constants.ts:23`
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
34:   REMIND_PASSWORD_SUCCESS: `${prefix}/REMIND_PASSWORD_SUCCESS`,
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
```

### `src/state/user/constants.ts:24`
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
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
```

### `src/state/user/constants.ts:25`
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
36: 
37:   SET_TAB: `${prefix}/SET_TAB`,
38:   SET_STATUS: `${prefix}/SET_STATUS`,
39:   SET_MESSAGE: `${prefix}/SET_MESSAGE`,
```

### `src/state/user/constants.ts:43`
```text
35:   REMIND_PASSWORD_FAIL: `${prefix}/REMIND_PASSWORD_FAIL`,
36: 
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
56: 
57: export const LOGIN_TABS = [
```

### `src/state/user/constants.ts:57`
```text
49: } as const
50: 
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
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
```

### `src/state/user/constants.ts:58`
```text
50: 
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
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
```

### `src/state/user/constants.ts:61`
```text
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
70:   tab: TLoginTab,
71:   response: IRegisterResponse | null,
72:   whatsappSignUpData?:{u_phone: string}|null,
73: }
```

### `src/state/user/constants.ts:63`
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

### `src/state/user/constants.ts:67`
```text
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

### `src/state/user/constants.ts:70`
```text
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

### `src/state/user/helpers.ts:6`
```text
1: import * as API from './../../API'
2: 
3: export function uploadFiles(data: any) {
4:   const filesMap: Record<string, any[]> = data.files
5:   const additionalDetails: any = data.u_details
6:   const params: any = data.tokens
7:   const uploads = Object.keys(filesMap)
8:     .filter(key => Array.isArray(filesMap[key]))
9:     .map(key => {
10:       const files = filesMap[key]
11:         .filter((item: [any, File]) => !item[0])
12:         .map((item: [any, File]) => item[1])
13:       const promises = files.map((file: File) => API.uploadFile({ file, ...params }))
14:       return Promise.all(promises).then(
15:         (res: any[]) => res.reduce((acc, item) => ({
16:           [key]: [...acc[key], item.data?.data?.dl_id],
17:         }), { [key]: [] }),
18:       ).then((res: any) => ({ [key]: JSON.stringify(res[key]) }))
19:     })
20: 
```

### `src/state/user/helpers.ts:26`
```text
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
```

### `src/state/user/helpers.ts:27`
```text
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
```

### `src/state/user/helpers.ts:63`
```text
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
14:   whatsappSignUpData: { u_phone: '' },
15: })
```

### `src/state/user/reducer.ts:9`
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
```

### `src/state/user/reducer.ts:12`
```text
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
22:       return state
23:         .set('status', EStatuses.Loading)
24:         .set('message', '')
25:         .set('user', null)
26:         .set('tokens', null)
```

## 3. Правило закрытия Auth bridge

Claim `Frontend → AUTHENTICATES_WITH → Core Backend` получает `CONFIRMED` только при наличии:

```text
конкретный frontend call
       ↓
конкретный backend endpoint
       ↓
конкретный response credential
       ↓
frontend storage/state
       ↓
обычный API request
       ↓
credential injection
```

## 4. Нельзя смешивать

```text
authentication
    ≠
authorization
    ≠
role-dependent UI
```

Наличие `id_role` в frontend не означает, что frontend самостоятельно выполняет backend authorization.

Backend RP-20 уже показал:

```text
API request → check_auth_user() → authenticated context
```

RP-33 должен только установить, какие frontend credentials приходят в этот request.

## 5. Current graph boundary

До закрытия concrete value-flow сохраняются:

```text
Taxi Frontend Snapshot
    → HAS_AUTH_IMPLEMENTATION

Core Backend
    → AUTHENTICATES_API_REQUEST
```

Не добавляется:

```text
Frontend token
    → IDENTICAL_TO
    → Backend auth credential
```

без direct evidence.

## 6. Gap Report

```text
G-33-01  exact frontend auth endpoint        OPEN
G-33-02  backend response credential          OPEN
G-33-03  credential persistence               OPEN
G-33-04  ordinary request injection           OPEN
G-33-05  value-flow into check_auth_user       OPEN
```

## 7. Следующий шаг

Если в source найден конкретный login endpoint, следующий pass должен открыть его backend implementation и frontend response handler вместе.

Цель:

```text
Frontend login
    ↓
Core Backend login endpoint
    ↓
response
    ↓
token/u_hash/user identity
    ↓
frontend state
    ↓
GET /query or another protected API
    ↓
check_auth_user()
```

Если login endpoint в данном frontend snapshot отсутствует, фиксируем `SOURCE_GAP`, а не реконструируем его по token usages.
