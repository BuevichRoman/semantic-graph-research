# Backend Semantic Graph — Research Pass 36
# Taxi Auth Exact Endpoint Match v0.1

**Статус:** EVIDENCE-GROUNDED / PROVISIONAL
**Методология:** Semantic Graph Research Methodology v2.3
**Предшествующий проход:** RP-35 v0.2 Taxi Authentication Frontend ↔ Core Backend Trace
**Источники:** `taxi-master.zip` + `archive_17012026_1259_clear.zip`

## 1. Research Question

> Какой exact backend route вызывается конкретным auth function Taxi Frontend и можно ли найти этот же route в Core Backend?

## 2. Frontend auth API contexts

### `src/API/car.ts:95`
```text
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
```

### `src/API/car.ts:184`
```text
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

### `src/API/user.ts:30`
```text
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
```

### `src/API/user.ts:34`
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
```

### `src/API/user.ts:110`
```text
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

### `src/API/auth.ts:37`
```text
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
```

### `src/API/auth.ts:51`
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
```

### `src/API/auth.ts:53`
```text
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

### `src/API/auth.ts:98`
```text
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

### `src/API/auth.ts:103`
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
```

### `src/API/auth.ts:108`
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

### `src/API/auth.ts:140`
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
```

### `src/API/auth.ts:142`
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

### `src/API/auth.ts:155`
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

### `src/API/auth.ts:174`
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
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
```

### `src/API/auth.ts:178`
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

### `src/API/auth.ts:193`
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

### `src/API/auth.ts:202`
```text
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

### `src/API/index.ts:35`
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

### `src/API/index.ts:41`
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
```

### `src/API/index.ts:96`
```text
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
```

### `src/API/index.ts:108`
```text
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
```

### `src/API/index.ts:116`
```text
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
```

### `src/API/index.ts:175`
```text
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
```

## 3. Frontend request construction


- `src/API/car.ts:1` — `import axios from 'axios'`
- `src/API/car.ts:5` — `import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'`
- `src/API/car.ts:26` — `export const createCar = apiMethod(async(`
- `src/API/car.ts:27` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:35` — `await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)`
- `src/API/car.ts:40` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:43` — `return axios.post(`${Config.API_URL}/car/${id}`, formData)`
- `src/API/car.ts:47` — `export const getCar = apiMethod<typeof _getCar>(_getCar)`
- `src/API/car.ts:50` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:53` — `return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)`
- `src/API/car.ts:57` — `export const getCars = apiMethod<typeof _getCars>(_getCars)`
- `src/API/car.ts:59` — `export const createUserCar = apiMethod(async(`
- `src/API/car.ts:60` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:67` — `const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)`
- `src/API/car.ts:73` — `const setDefaultCarLicenses = apiMethod(async(`
- `src/API/car.ts:74` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:89` — `await axios.post(`${Config.API_URL}/car/${id}`, formData)`
- `src/API/car.ts:93` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:95` — `return axios.post(`${Config.API_URL}/user/authorized/car`, formData)`
- `src/API/car.ts:98` — `export const getUserCars = apiMethod<typeof _getUserCars>(_getUserCars)`
- `src/API/car.ts:101` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:108` — `return axios.post(`${Config.API_URL}/user/${id}/car`, formData)`
- `src/API/car.ts:112` — `export const getUserCar = apiMethod<typeof _getUserCar>(_getUserCar)`
- `src/API/car.ts:114` — `export const editCar = apiMethod(async(`
- `src/API/car.ts:115` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:128` — `const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)`
- `src/API/car.ts:132` — `export const driveCar = apiMethod(async(`
- `src/API/car.ts:133` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:138` — `let { data: response } = await axios.post(`
- `src/API/car.ts:139` — ``${Config.API_URL}/car/${car.c_id}/drive`,`
- `src/API/car.ts:160` — `const syncUserWithCar = apiMethod(async(`
- `src/API/car.ts:161` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:172` — `await axios.post(`${Config.API_URL}/user`, formData)`
- `src/API/car.ts:181` — `{ formData }: IApiMethodArguments,`
- `src/API/car.ts:183` — `const { data } = await axios.post(`
- `src/API/car.ts:184` — ``${Config.API_URL}/user/authorized/car/driven`,`
- `src/API/car.ts:190` — `apiMethod<typeof _getUserDrivenCar>(_getUserDrivenCar)`
- `src/API/user.ts:1` — `import axios from 'axios'`
- `src/API/user.ts:4` — `import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'`
- `src/API/user.ts:8` — `{ formData }: IApiMethodArguments,`
- `src/API/user.ts:11` — `return axios.post(`${Config.API_URL}/user/${id}`, formData)`
- `src/API/user.ts:15` — `export const getUser = apiMethod<typeof _getUser>(_getUser)`
- `src/API/user.ts:18` — `{ formData }: IApiMethodArguments,`
- `src/API/user.ts:21` — `return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)`
- `src/API/user.ts:25` — `export const getUsers = apiMethod<typeof _getUsers>(_getUsers)`
- `src/API/user.ts:28` — `{ formData }: IApiMethodArguments,`
- `src/API/user.ts:30` — `return axios.post(`${Config.API_URL}/user/authorized`, formData)`
- `src/API/user.ts:34` — `export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)`
- `src/API/user.ts:86` — `{ formData }: IApiMethodArguments,`
- `src/API/user.ts:106` — `return axios.post(`${Config.API_URL}/user`, formData)`
- `src/API/user.ts:109` — `export const editUser = apiMethod<typeof _editUser>(_editUser)`
- `src/API/user.ts:110` — `export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })`
- `src/API/auth.ts:1` — `import axios from 'axios'`
- `src/API/auth.ts:4` — `import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'`
- `src/API/auth.ts:9` — `{ formData }: IApiMethodArguments,`
- `src/API/auth.ts:19` — `return axios.post(`${Config.API_URL}/register`, formData)`
- `src/API/auth.ts:37` — `export const register = apiMethod<typeof _register>(_register, { authRequired: false })`
- `src/API/auth.ts:40` — `{ formData }: IApiMethodArguments,`
- `src/API/auth.ts:47` — `return axios.post(`${Config.API_URL}/remind`, formData)`
- `src/API/auth.ts:51` — `export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })`
- `src/API/auth.ts:54` — `{ formData }: IApiMethodArguments,`
- `src/API/auth.ts:66` — `return axios.post(`${Config.API_URL}/auth`, formData)`
- `src/API/auth.ts:105` — `return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })`
- `src/API/auth.ts:117` — `export const login = apiMethod<typeof _login>(_login, { authRequired: false })`
- `src/API/auth.ts:120` — `_: IApiMethodArguments,`
- `src/API/auth.ts:133` — `return axios.post(`${Config.API_URL}/register`, waData)`
- `src/API/auth.ts:140` — `export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })`
- `src/API/auth.ts:143` — `{ formData }: IApiMethodArguments,`
- `src/API/auth.ts:163` — `return axios.post(`${Config.API_URL}/register`, formData)`
- `src/API/auth.ts:174` — `return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })`
- `src/API/auth.ts:189` — `return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })`
- `src/API/auth.ts:202` — `export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })`
- `src/API/auth.ts:205` — `{ formData }: IApiMethodArguments,`
- `src/API/auth.ts:207` — `return axios.post(`${Config.API_URL}/logout/?`)`
- `src/API/auth.ts:209` — `export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })`
- `src/API/index.ts:1` — `import axios from 'axios'`
- `src/API/index.ts:18` — `addToFormData, apiMethod, IApiMethodArguments, IResponseFields,`
- `src/API/index.ts:57` — `postDrive,`
- `src/API/index.ts:77` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:92` — `.then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))`
- `src/API/index.ts:96` — `export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })`
- `src/API/index.ts:99` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:102` — `return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)`
- `src/API/index.ts:108` — `export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })`
- `src/API/index.ts:111` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:114` — `return axios.get(`${Config.API_URL}`, { params: { config } })`
- `src/API/index.ts:116` — `export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })`
- `src/API/index.ts:118` — `const _postTrip = (`
- `src/API/index.ts:119` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:128` — `return axios.post(`${Config.API_URL}/trip`, formData)`
- `src/API/index.ts:131` — `export const postTrip = apiMethod<typeof _postTrip>(_postTrip)`
- `src/API/index.ts:134` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:141` — `return axios.post(`${Config.API_URL}/trip`, formData)`
- `src/API/index.ts:153` — `export const getTrips = apiMethod<typeof _getTrips>(_getTrips)`
- `src/API/index.ts:156` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:163` — `return axios.post(`${Config.API_URL}/trip/get`, formData)`
- `src/API/index.ts:175` — `export const getWashTrips = apiMethod<typeof _getWashTrips>(_getWashTrips, { authRequired: false })`
- `src/API/index.ts:178` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:181` — `return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {`
- `src/API/index.ts:184` — `return [id, URL.createObjectURL(res.data)]`
- `src/API/index.ts:187` — `export const getImageBlob = apiMethod<typeof _getImageBlob>(_getImageBlob)`
- `src/API/index.ts:190` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:193` — `return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {`
- `src/API/index.ts:199` — `export const getImageFile = apiMethod<typeof _getImageFile>(_getImageFile)`
- `src/API/index.ts:202` — `{ formData }: IApiMethodArguments,`
- `src/API/index.ts:233` — `return axios.post(`${Config.API_URL}/user`, formData)`
- `src/API/index.ts:236` — `export const setOutDrive = apiMethod<typeof _setOutDrive>(_setOutDrive)`
- `src/API/index.ts:247` — `return axios.get(`
- `src/API/index.ts:270` — `return axios.get(`
- `src/API/index.ts:293` — `return axios.get(`
- `src/API/index.ts:322` — `axios.post('http://jecat.ru/car_api/api/notifypos.php', {`
- `src/API/index.ts:384` — `const officialSuggestions = await axios.get(`
- `src/API/index.ts:426` — `return axios.get('https://chat.itest24.com/wschat/checksrv.php', {`

## 4. Backend auth route/controller candidates


### `archive_17012026_1259/taxi/index.php:7`
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
29: 			}
30: 			else
31: 			{
32: 				session_write_close();
```

### `archive_17012026_1259/taxi/index.php:14`
```php
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
29: 			}
30: 			else
31: 			{
32: 				session_write_close();
33: 				unset($_SESSION);
34: 				$_SESSION[UID] = $AUID;
35: 			}
36: 		}
37: 	}
38: 	require_once('models/PHPMailer/class.smtp.php');
39: 	require_once('models/PHPMailer/class.phpmailer.php');
```

### `archive_17012026_1259/taxi/index.php:17`
```php
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
29: 			}
30: 			else
31: 			{
32: 				session_write_close();
33: 				unset($_SESSION);
34: 				$_SESSION[UID] = $AUID;
35: 			}
36: 		}
37: 	}
38: 	require_once('models/PHPMailer/class.smtp.php');
39: 	require_once('models/PHPMailer/class.phpmailer.php');
40: 
41: 	require_once('models/m_functions.php');
42: 	if (isset($_REQUEST['lang']) && !empty(taxi::$data['langs'][$_REQUEST['lang'] = trim($_REQUEST['lang'])])) {$_SESSION['lang'] = $_REQUEST['lang'];} else {unset($_REQUEST['lang']);}
```

### `archive_17012026_1259/taxi/models/api.php:30`
```php
20: 			{
21: 				if (empty($_SESSION[UID])) 
22: 				{
23: 					if ($role != 1 && $role != 2 && $role != 5)
24: 					{
25: 						return $this->showError('404', 'error', 'wrong user role');
26: 					}
27: 				}
28: 				else
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
32: 					$sql_user = "'" . $_SESSION[UID] . "'";
33: 				}
34: 			}
35: 			else
36: 			{
37: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
38: 				if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
39: 				$sql_user = "'" . $_SESSION[UID] . "'";
40: 			}
41: 
42: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa)) 
43: 			{
44: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
45: 			} 
46: 			else
47: 			{
48: 				$sql_email = "`email` = '" . $email . "'";
49: 				$sql_phone = "`phone` = '" . preg_replace('/[^0-9]+/','',$phone) . "'";
50: 				$sql_tg = "`tg` = '" . $tg . "'";
51: 				$sql_wa = "`wa` = '" . $wa . "'";				
52: 				
53: 				$sql_field = array();
54: 				$sql_where = array();
55: 				if (empty($phone)) 
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
359: 						`id_role`,
360: 						`id_user`,
361: 						`name`,
362: 						`family`,
363: 						`middle`,
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
362: 						`family`,
363: 						`middle`,
364: 						`phone`,
365: 						`phone_is_verified`,
366: 						`email`,
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
365: 						`phone_is_verified`,
366: 						`email`,
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
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
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
```

### `archive_17012026_1259/taxi/models/api.php:348`
```php
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
367: 						`photo_link`,
368: 						`pwd`,
369: 						`id_lang`,
370: 						`currency`,
371: 						`id_city`,
372: 						`tips`,
373: 						`language_skills`,
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
376: 						`id_verification_status`,
377: 						`active`,
378: 						`deleted`,
379: 						`birthday_date`
380: 					FROM `users`
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
413: 				if (!empty($send_par['error'])) return $this->showError('404', 'error', $send_par['error']);
414: 				if (empty($type_arr[1]))
415: 				{
416: 					$type = $send_par['type'];
417: 					
```

### `archive_17012026_1259/taxi/models/api.php:403`
```php
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
417: 					
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
```

### `archive_17012026_1259/taxi/models/api.php:412`
```php
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
417: 					
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
```

### `archive_17012026_1259/taxi/models/api.php:419`
```php
409: 			{
410: 				$type_arr = explode(',',$type);
411: 				$type_code = $type_arr[0];
412: 				$send_par = $this->getSendParForAuthCode($type_code);
413: 				if (!empty($send_par['error'])) return $this->showError('404', 'error', $send_par['error']);
414: 				if (empty($type_arr[1]))
415: 				{
416: 					$type = $send_par['type'];
417: 					
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
```

### `archive_17012026_1259/taxi/models/api.php:421`
```php
411: 				$type_code = $type_arr[0];
412: 				$send_par = $this->getSendParForAuthCode($type_code);
413: 				if (!empty($send_par['error'])) return $this->showError('404', 'error', $send_par['error']);
414: 				if (empty($type_arr[1]))
415: 				{
416: 					$type = $send_par['type'];
417: 					
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
449: 				}
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
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
450: 				elseif (!empty($d['deleted']))
451: 				{
452: 					return $this->showError('404', 'error', 'phone of deleted user');
453: 				}
454: 				$user_ban_status = get_user_ban_status($d['id_user']);
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
456: 				{
457: 					return $this->showError('404', 'error', 'phone of banned user');
458: 				}
459: 				$send_par['login_val'] = $d[$sql_login];
460: 
```

### `archive_17012026_1259/taxi/models/api.php:455`
```php
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
473: 								case 'id':
474: 									return $this->showError('404', 'error', 'whatsapp id not supported');
475: 									break;
476: 							}
477: 							break;
478: 						case'mail':
479: 							$dataForEmail = array(
480: 								'{$u_id}' 			=> $d['id_user'],
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
480: 								'{$u_id}' 			=> $d['id_user'],
481: 								'{$code}' 			=> $code,			
482: 								'{$locationPath}' 	=> url('',CONFIG_URL)
483: 							);
484: 							$subject = lang('email_code_subject','Code',2);
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
489: 						case 'sms':
490: 							$sms_status = send_sms($send_par['login_val'],$code);
491: 							if (!empty($sms_status['error'])) return $this->showError('404', 'error', $sms_status['error']);
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
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
492: 							$json_str = real_escape_string(json_encode(array('sms_id'=>$sms_status)));
493: 							break;
494: 						case 'telegram':
495: 							switch ($send_par['login'])
496: 							{
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
508: 					}
509: 
510: 					$s = "INSERT INTO `users_code`
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
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
511: 						SET 
512: 							`id_user` = '" . $d['id_user'] . "',
513: 							`code` = '" . $code . "',
514: 							`expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "',
515: 							`auth_type` = '$type_code,$type',
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
516: 							`json` = '$json_str'
517: 						ON DUPLICATE KEY UPDATE 
518: 							`code` = '" . $code . "', `expire_datetime` = '" . (time() + $this->constant['auth_code_interval']) . "', `json` = '$json_str'
519: 						";
520: 
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
522: 					if ($q === false) return $this->showError('404', 'error', 'database insert failed');
523: 					return array(
524: 							'code' 		=>	'200',
525: 							'status' 	=>	'success',
526: 							'data'		=>	'code sent'
```

### `archive_17012026_1259/taxi/models/api.php:514`
```php
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
522: 					if ($q === false) return $this->showError('404', 'error', 'database insert failed');
523: 					return array(
524: 							'code' 		=>	'200',
525: 							'status' 	=>	'success',
526: 							'data'		=>	'code sent'
527: 						);
528: 				}
529: 				else
530: 				{
531: 					$password = preg_replace('/[^0-9]+/','',$password);
532: 
533: 					$s = "SELECT 
534: 						`users_code`.`id_user`,
535: 						IF(`users_code`.`expire_datetime` > '" . time() . "',0,1) as active,
536: 						`users`.`id_role`,
537: 						`users`.`id_user`,
538: 						`users`.`name`,
539: 						`users`.`family`,
```

### `archive_17012026_1259/taxi/models/api.php:515`
```php
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
522: 					if ($q === false) return $this->showError('404', 'error', 'database insert failed');
523: 					return array(
524: 							'code' 		=>	'200',
525: 							'status' 	=>	'success',
526: 							'data'		=>	'code sent'
527: 						);
528: 				}
529: 				else
530: 				{
531: 					$password = preg_replace('/[^0-9]+/','',$password);
532: 
533: 					$s = "SELECT 
534: 						`users_code`.`id_user`,
535: 						IF(`users_code`.`expire_datetime` > '" . time() . "',0,1) as active,
536: 						`users`.`id_role`,
537: 						`users`.`id_user`,
538: 						`users`.`name`,
539: 						`users`.`family`,
540: 						`users`.`middle`,
```

### `archive_17012026_1259/taxi/models/api.php:518`
```php
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
522: 					if ($q === false) return $this->showError('404', 'error', 'database insert failed');
523: 					return array(
524: 							'code' 		=>	'200',
525: 							'status' 	=>	'success',
526: 							'data'		=>	'code sent'
527: 						);
528: 				}
529: 				else
530: 				{
531: 					$password = preg_replace('/[^0-9]+/','',$password);
532: 
533: 					$s = "SELECT 
534: 						`users_code`.`id_user`,
535: 						IF(`users_code`.`expire_datetime` > '" . time() . "',0,1) as active,
536: 						`users`.`id_role`,
537: 						`users`.`id_user`,
538: 						`users`.`name`,
539: 						`users`.`family`,
540: 						`users`.`middle`,
541: 						`users`.`phone`,
542: 						`users`.`phone_is_verified`,
543: 						`users`.`email`,
```

### `archive_17012026_1259/taxi/models/api.php:560`
```php
550: 						`users`.`language_skills`,
551: 						`users`.`description`,
552: 						`users`.`id_navigation`,
553: 						`users`.`id_verification_status`,
554: 						`users`.`active`,
555: 						`users`.`birthday_date`				
556: 					FROM `users_code`					
557: 					LEFT JOIN `users` USING (`id_user`)
558: 					WHERE 
559: 						`users_code`.`id_user` = '" . $d['id_user'] . "' AND 
560: 						`auth_type` = '$type_code,$type' AND 
561: 						`users_code`.`code` = '" . $password . "' 
562: 					LIMIT 1
563: 					";
564: 
565: 					$q = query($s);
566: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
567: 					$d = fetch_assoc($q);
568: 					if (empty($d['id_user']))
569: 					{
570: 						return $this->showError('404', 'error', 'wrong code');
571: 					}
572: 					if (empty($d['active']))
573: 					{
574: 						return $this->showError('404', 'error', 'expired code');
575: 					}
576: 				}
577: 			}
578: 
579: 			$_SESSION['auth_time'] = time();
580: 			$_SESSION[UID] = $d['id_user'];
581: 			$_SESSION['name'] = $d['name'];
582: 			$_SESSION['family'] = $d['family'];
583: 			$_SESSION['middle'] = $d['middle'];
584: 			$_SESSION['email'] = $d['email'];
585: 			$_SESSION['phone'] = $d['phone'];
```

### `archive_17012026_1259/taxi/models/api.php:579`
```php
569: 					{
570: 						return $this->showError('404', 'error', 'wrong code');
571: 					}
572: 					if (empty($d['active']))
573: 					{
574: 						return $this->showError('404', 'error', 'expired code');
575: 					}
576: 				}
577: 			}
578: 
579: 			$_SESSION['auth_time'] = time();
580: 			$_SESSION[UID] = $d['id_user'];
581: 			$_SESSION['name'] = $d['name'];
582: 			$_SESSION['family'] = $d['family'];
583: 			$_SESSION['middle'] = $d['middle'];
584: 			$_SESSION['email'] = $d['email'];
585: 			$_SESSION['phone'] = $d['phone'];
586: 			$_SESSION['id_role'] = $d['id_role'];
587: 			$_SESSION['id_verification_status'] = $d['id_verification_status'];
588: 			$_SESSION['user_ban_status'] = $user_ban_status;
589: 			$_SESSION['active'] = (int)$d['active'];
590: 			$_SESSION['lang'] = $d['id_lang'] ? : (isset($_SESSION['lang']) ? $_SESSION['lang'] : DEFAULT_LANG);
591: 			$_SESSION['photo_link'] = $d['photo_link'] ? url($d['photo_link'],FILE_PATH) : '';
592: 			$_SESSION['birthday_date'] = $d['birthday_date'];
593: 			$_SESSION['id_lang'] = $d['id_lang'];
594: 			$_SESSION['currency'] = $d['currency'];
595: 			$_SESSION['id_navigation'] = $d['id_navigation'];
596: 			$vfoc = md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))));
597: 			setcookie("vfoc", $vfoc, 0, ROOT_URL);
598: 			$_SESSION['auth_hash'] = openssl____encrypt(session_id() . "|" . $vfoc);
599: 			return array(
600: 				'code' 		=>	'200',
601: 				'status' 	=>	'success',
602: 				'auth_user' =>	array(
603: 									'u_id' => $_SESSION[UID],
604: 									'u_name' => $_SESSION['name'],
```

### `archive_17012026_1259/taxi/models/api.php:598`
```php
588: 			$_SESSION['user_ban_status'] = $user_ban_status;
589: 			$_SESSION['active'] = (int)$d['active'];
590: 			$_SESSION['lang'] = $d['id_lang'] ? : (isset($_SESSION['lang']) ? $_SESSION['lang'] : DEFAULT_LANG);
591: 			$_SESSION['photo_link'] = $d['photo_link'] ? url($d['photo_link'],FILE_PATH) : '';
592: 			$_SESSION['birthday_date'] = $d['birthday_date'];
593: 			$_SESSION['id_lang'] = $d['id_lang'];
594: 			$_SESSION['currency'] = $d['currency'];
595: 			$_SESSION['id_navigation'] = $d['id_navigation'];
596: 			$vfoc = md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))));
597: 			setcookie("vfoc", $vfoc, 0, ROOT_URL);
598: 			$_SESSION['auth_hash'] = openssl____encrypt(session_id() . "|" . $vfoc);
599: 			return array(
600: 				'code' 		=>	'200',
601: 				'status' 	=>	'success',
602: 				'auth_user' =>	array(
603: 									'u_id' => $_SESSION[UID],
604: 									'u_name' => $_SESSION['name'],
605: 									'u_family' => $_SESSION['family'],
606: 									'u_middle' => $_SESSION['middle'],
607: 									'u_email' => $_SESSION['email'],
608: 									'u_phone' => $_SESSION['phone'],
609: 									'u_role' => $_SESSION['id_role'],
610: 									'u_a_role' => $this->id_role,
611: 									'u_check_state' => $_SESSION['id_verification_status'],
612: 									'u_ban' => $_SESSION['user_ban_status'],
613: 									'u_active' => $_SESSION['active'],
614: 									'u_photo' => $_SESSION['photo_link'],
615: 									'u_birthday' => $_SESSION['birthday_date'],
616: 									'u_phone_checked' => (int)$d['phone_is_verified'],
617: 									'u_lang' => $_SESSION['id_lang'],
618: 									'u_currency' => $_SESSION['currency'],
619: 									'u_city' => $d['id_city'],
620: 									'u_tips' => $d['tips'],
621: 									'u_lang_skills' => $d['language_skills'],
622: 									'u_description' => $d['description'],
623: 									'u_gps_software' => $_SESSION['id_navigation']
```

### `archive_17012026_1259/taxi/models/api.php:602`
```php
592: 			$_SESSION['birthday_date'] = $d['birthday_date'];
593: 			$_SESSION['id_lang'] = $d['id_lang'];
594: 			$_SESSION['currency'] = $d['currency'];
595: 			$_SESSION['id_navigation'] = $d['id_navigation'];
596: 			$vfoc = md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))));
597: 			setcookie("vfoc", $vfoc, 0, ROOT_URL);
598: 			$_SESSION['auth_hash'] = openssl____encrypt(session_id() . "|" . $vfoc);
599: 			return array(
600: 				'code' 		=>	'200',
601: 				'status' 	=>	'success',
602: 				'auth_user' =>	array(
603: 									'u_id' => $_SESSION[UID],
604: 									'u_name' => $_SESSION['name'],
605: 									'u_family' => $_SESSION['family'],
606: 									'u_middle' => $_SESSION['middle'],
607: 									'u_email' => $_SESSION['email'],
608: 									'u_phone' => $_SESSION['phone'],
609: 									'u_role' => $_SESSION['id_role'],
610: 									'u_a_role' => $this->id_role,
611: 									'u_check_state' => $_SESSION['id_verification_status'],
612: 									'u_ban' => $_SESSION['user_ban_status'],
613: 									'u_active' => $_SESSION['active'],
614: 									'u_photo' => $_SESSION['photo_link'],
615: 									'u_birthday' => $_SESSION['birthday_date'],
616: 									'u_phone_checked' => (int)$d['phone_is_verified'],
617: 									'u_lang' => $_SESSION['id_lang'],
618: 									'u_currency' => $_SESSION['currency'],
619: 									'u_city' => $d['id_city'],
620: 									'u_tips' => $d['tips'],
621: 									'u_lang_skills' => $d['language_skills'],
622: 									'u_description' => $d['description'],
623: 									'u_gps_software' => $_SESSION['id_navigation']
624: 								),
625: 				'auth_hash' => $_SESSION['auth_hash']
626: 			);
627: 		}
```

### `archive_17012026_1259/taxi/models/api.php:625`
```php
615: 									'u_birthday' => $_SESSION['birthday_date'],
616: 									'u_phone_checked' => (int)$d['phone_is_verified'],
617: 									'u_lang' => $_SESSION['id_lang'],
618: 									'u_currency' => $_SESSION['currency'],
619: 									'u_city' => $d['id_city'],
620: 									'u_tips' => $d['tips'],
621: 									'u_lang_skills' => $d['language_skills'],
622: 									'u_description' => $d['description'],
623: 									'u_gps_software' => $_SESSION['id_navigation']
624: 								),
625: 				'auth_hash' => $_SESSION['auth_hash']
626: 			);
627: 		}
628: 
629: 		public function logout()
630: 		{
631: 			unset($_SESSION[UID]);
632: 			session_unset();
633: 			return array(
634: 				'code' 		=>	'200',
635: 				'status' 	=>	'success'		
636: 			);
637: 		}
638: 		
639: 		public function selectUser($id_user = "", $users_props = array(), $field_types = array())
640: 		{
641: 			if (empty($_SESSION[UID])) {
642: 				$prop_visibility = 1;
643: 				return $this->showError('404', 'error', 'unauthorized access');
644: 			}
645: 			$sql_add = "
646: 					`phone` as u_phone,
647: 					`phone_is_verified` as u_phone_checked,
648: 					`email` as u_email,
649: 					`email_is_verified` as u_email_checked,
650: 					`birthday_date` as u_birthday,
```

### `archive_17012026_1259/taxi/models/api.php:643`
```php
633: 			return array(
634: 				'code' 		=>	'200',
635: 				'status' 	=>	'success'		
636: 			);
637: 		}
638: 		
639: 		public function selectUser($id_user = "", $users_props = array(), $field_types = array())
640: 		{
641: 			if (empty($_SESSION[UID])) {
642: 				$prop_visibility = 1;
643: 				return $this->showError('404', 'error', 'unauthorized access');
644: 			}
645: 			$sql_add = "
646: 					`phone` as u_phone,
647: 					`phone_is_verified` as u_phone_checked,
648: 					`email` as u_email,
649: 					`email_is_verified` as u_email_checked,
650: 					`birthday_date` as u_birthday,
651: 					`referral_code` as ref_code,
652: 					`referrer` as referrer_u_id,
653: 					`tg` as u_tg,
654: 					`tg_is_verified` as u_tg_checked,
655: 					`id_user_upper` as u_upper,
656: 					`wa` as u_wa,
657: 					`wa_is_verified` as u_wa_checked,";
658: 			$sql_limit = "";
659: 			if (empty($id_user))
660: 			{	
661: 				if ($this->id_role != 4)
662: 				{
663: 					$prop_visibility = 4;
664: 					$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
665: 					$sql_limit = "LIMIT 1";
666: 				}
667: 				else
668: 				{
```

### `archive_17012026_1259/taxi/models/api.php:676`
```php
666: 				}
667: 				else
668: 				{
669: 					$prop_visibility = 8;
670: 					$sql_where = "";
671: 					$sql_limit = "LIMIT " . $this->limit_offset . ", " . $this->limit_row_count;
672: 				}
673: 			}
674: 			else
675: 			{	
676: 				if ($id_user == 'authorized' || $id_user == $_SESSION[UID]) 
677: 				{
678: 					$prop_visibility = 4;
679: 					$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
680: 					$sql_limit = "LIMIT 1";
681: 				}
682: 				else
683: 				{
684: 					$prop_visibility = 8;
685: 					if ($this->id_role != 4) 
686: 					{
687: 						$sql_add = "";
688: 						$prop_visibility = 2;
689: 					}
690: 					$sql_where = "AND `id_user` in (" . $id_user . ")";
691: 					$sql_limit = "LIMIT " . $this->constant['limit_row_count_max'];
692: 				}
693: 			}
694: 
695: 			$sql_props = array();
696: 			$find_props = array();
697: 			foreach($users_props as $id_users_prop=>$prop_arr)
698: 			{
699: 				if ($prop_arr['visibility'] & $prop_visibility)
700: 				{
701: 					$value_type_str = $field_types[$prop_arr['value_type']]['var'];
```

### `archive_17012026_1259/taxi/models/api.php:843`
```php
833: 					unset($d[$find_prop]);
834: 				}
835: 
836: 				$out['user'][$d['u_id']] = $d;
837: 			}
838: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
839: 			return array(
840: 				'code' 		=>	'200',
841: 				'status' 	=>	'success',		
842: 				'data' 		=>	$out,
843: 				'auth_user' =>	array(
844: 									'u_id' => $_SESSION[UID],
845: 									'u_name' => $_SESSION['name'],
846: 									'u_family' => $_SESSION['family'],
847: 									'u_middle' => $_SESSION['middle'],
848: 									'u_email' => $_SESSION['email'],
849: 									'u_phone' => $_SESSION['phone'],
850: 									'u_role' => $_SESSION['id_role'],
851: 									'u_a_role' => $this->id_role,
852: 									'u_check_state' => $_SESSION['id_verification_status'],
853: 									'u_ban' => $_SESSION['user_ban_status'],
854: 									'u_active' => $_SESSION['active'],
855: 									'u_photo' => $_SESSION['photo_link'],
856: 									'u_birthday' => $_SESSION['birthday_date'],
857: 									'u_lang' => $_SESSION['id_lang'],
858: 									'u_currency' => $_SESSION['currency'],
859: 									'u_gps_software' => $_SESSION['id_navigation']
860: 								)
861: 			);
862: 		}
863: 
864: 		public function editUser($data = "", $id_user = "", $roles = array(), $users_props = array(), $field_types = array())
865: 		{
866: 			if (empty($_SESSION[UID])) {
867: 				return $this->showError('404', 'error', 'unauthorized access');
868: 			}	
```

### `archive_17012026_1259/taxi/models/api.php:867`
```php
857: 									'u_lang' => $_SESSION['id_lang'],
858: 									'u_currency' => $_SESSION['currency'],
859: 									'u_gps_software' => $_SESSION['id_navigation']
860: 								)
861: 			);
862: 		}
863: 
864: 		public function editUser($data = "", $id_user = "", $roles = array(), $users_props = array(), $field_types = array())
865: 		{
866: 			if (empty($_SESSION[UID])) {
867: 				return $this->showError('404', 'error', 'unauthorized access');
868: 			}	
869: 			if (empty($data)) 
870: 			{
871: 				return $this->showError('404', 'error', 'empty data');
872: 			}
873: 
874: 			$data = json_decode($data,true);
875: 			
876: 			if (empty($data) || !is_array($data)) 
877: 			{
878: 				return $this->showError('404', 'error', 'wrong data');
879: 			}
880: 
881: 			$allowed_fields = array(
882: 									'u_role'				=>		'id_role',
883: 									'u_name'				=>		'name',
884: 									'u_family'				=>		'family',
885: 									'u_middle'				=>		'middle',
886: 									'u_phone'				=>		'phone',
887: 									'u_phone_checked'		=>		'phone_is_verified',
888: 									'u_email'				=>		'email',
889: 									'u_email_checked'		=>		'email_is_verified',
890: 									'u_photo'				=>		'photo_link',
891: 									'u_lang'				=>		'id_lang',
892: 									'u_currency'			=>		'currency',
```

### `archive_17012026_1259/taxi/models/api.php:1348`
```php
1338: 				if ($details_is_updated === true)
1339: 				{
1340: 					$data['u_details'] = real_escape_string(json_encode($details));
1341: 				}
1342: 				else
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

### `archive_17012026_1259/taxi/models/api.php:1349`
```php
1339: 				{
1340: 					$data['u_details'] = real_escape_string(json_encode($details));
1341: 				}
1342: 				else
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
1374: 			{
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
1374: 			{
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
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
1375: 				$u_login = "u_$login";
1376: 				if ($data[$u_login] !== $$login)
1377: 				{
1378: 					if ($data[$u_login] === "") return $this->showError('404', 'error', "empty user $login");
1379: 					if ($this->id_role != 4)
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
1379: 					if ($this->id_role != 4)
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
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
1380: 					{
1381: 						$key = "u_{$login}_checked";
1382: 						$affected_fields[] = $key;
1383: 						$affected_keys[$key] = true;
1384: 						$data[$key] = 0;
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
1384: 						$data[$key] = 0;
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
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
1385: 						$allowed_fields[$key] = "{$login}_is_verified";
1386: 					}
1387: 					if ($data[$u_login] !== NULL) 
1388: 					{
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
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
1389: 						$sql_where_str = "`$login` = '{$data[$u_login]}'";
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
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
1390: 						$sql_field[] = "COUNT(IF($sql_where_str,1,NULL)) as {$login_flags[$i]}";
1391: 						$sql_where[] = $sql_where_str;
1392: 					}
1393: 					$$login = $data[$u_login];
1394: 				}
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
1394: 				}
1395: 				else
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
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
1396: 				{
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
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
1397: 					unset($affected_keys[$u_login]);
1398: 				}
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
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
1399: 			}
1400: 
1401: 			if (empty($phone) && empty($email) && empty($tg) && empty($wa))
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
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
1402: 			{
1403: 				return $this->showError('404', 'error', 'empty user phone and email and tg and wa');
1404: 			}
1405: 			if (!empty($sql_where))
1406: 			{
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
1406: 			{
1407: 				$sql_field = implode(",\n", $sql_field);
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
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
1408: 				$sql_where = implode(" OR ", $sql_where);
1409: 				$s = "SELECT 
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
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
1410: 						$sql_field 
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
1414: 
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
1411: 					FROM `users`
1412: 					WHERE 
1413: 						$sql_where";
1414: 
1415: 				$q = query($s);
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
1414: 
1415: 				$q = query($s);
1416: 				if ($q === false) return $this->showError('404', 'error', 'mysql select failed');
1417: 				$d = fetch_assoc($q);
1418: 				$msg = array();
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
1418: 				$msg = array();
1419: 				foreach($login_flags as $i=>$login_flag)
1420: 				{
1421: 					if (!empty($d[$login_flag]))
1422: 					{
```

### `archive_17012026_1259/taxi/models/api.php:1419`
```php
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
1424: 					}
1425: 				}
1426: 				if (!empty($msg))return $this->showError('404', 'error', 'busy user data: ' . implode(", ", $msg));
1427: 			}
1428: 			
1429: 			if (!empty($affected_keys['ref_code']))
1430: 			{
1431: 				if (strtolower(substr($data['ref_code'],0,3)) == "uid" && $data['ref_code'] != "uid$id_user")
1432: 				{
1433: 					return $this->showError('404', 'error', 'uid ref_code fot other user');
1434: 				}
1435: 				if ($data['ref_code'] === "")
1436: 				{
1437: 					return $this->showError('404', 'error', 'empty ref_code string');
1438: 				}
1439: 				elseif ($data['ref_code'] !== NULL)
1440: 				{
1441: 					$s = "SELECT 
1442: 							`id_user` 
1443: 						FROM `users`
1444: 						WHERE `referral_code` = '" . $data['ref_code'] . "'
```

### `archive_17012026_1259/taxi/models/api.php:1421`
```php
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
1424: 					}
1425: 				}
1426: 				if (!empty($msg))return $this->showError('404', 'error', 'busy user data: ' . implode(", ", $msg));
1427: 			}
1428: 			
1429: 			if (!empty($affected_keys['ref_code']))
1430: 			{
1431: 				if (strtolower(substr($data['ref_code'],0,3)) == "uid" && $data['ref_code'] != "uid$id_user")
1432: 				{
1433: 					return $this->showError('404', 'error', 'uid ref_code fot other user');
1434: 				}
1435: 				if ($data['ref_code'] === "")
1436: 				{
1437: 					return $this->showError('404', 'error', 'empty ref_code string');
1438: 				}
1439: 				elseif ($data['ref_code'] !== NULL)
1440: 				{
1441: 					$s = "SELECT 
1442: 							`id_user` 
1443: 						FROM `users`
1444: 						WHERE `referral_code` = '" . $data['ref_code'] . "'
1445: 						LIMIT 1";
1446: 
```

### `archive_17012026_1259/taxi/models/api.php:1423`
```php
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
1424: 					}
1425: 				}
1426: 				if (!empty($msg))return $this->showError('404', 'error', 'busy user data: ' . implode(", ", $msg));
1427: 			}
1428: 			
1429: 			if (!empty($affected_keys['ref_code']))
1430: 			{
1431: 				if (strtolower(substr($data['ref_code'],0,3)) == "uid" && $data['ref_code'] != "uid$id_user")
1432: 				{
1433: 					return $this->showError('404', 'error', 'uid ref_code fot other user');
1434: 				}
1435: 				if ($data['ref_code'] === "")
1436: 				{
1437: 					return $this->showError('404', 'error', 'empty ref_code string');
1438: 				}
1439: 				elseif ($data['ref_code'] !== NULL)
1440: 				{
1441: 					$s = "SELECT 
1442: 							`id_user` 
1443: 						FROM `users`
1444: 						WHERE `referral_code` = '" . $data['ref_code'] . "'
1445: 						LIMIT 1";
1446: 
1447: 					$q = query($s);
1448: 					if ($q === false) return $this->showError('404', 'error', 'mysql ref_code select failed');
```

### `archive_17012026_1259/taxi/models/api.php:1853`
```php
1843: 					'code' 		=>	'200',
1844: 					'status' 	=>	'success',
1845: 					'data'		=>	$res
1846: 				);
1847: 			}
1848: 			return $this->showError('404', 'error', 'wrong hash');
1849: 		}
1850: 
1851: 		public function selectCar($id_car = "", $id_user = "", $langs = array())
1852: 		{
1853: 			if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
1854: 			$sql = $sql_json = $sql_left_join = "";
1855: 			if (!empty($id_user))
1856: 			{
1857: 				if ($id_user == 'authorized') 
1858: 				{
1859: 					if (empty($id_car))
1860: 					{
1861: 						$sql = "";
1862: 					}
1863: 					elseif ($id_car == 'driven')
1864: 					{
1865: 						$sql = "AND `used` = '1'";
1866: 					}
1867: 					else
1868: 					{
1869: 						$sql = "AND `id_car` in (" . $id_car . ")";
1870: 					}
1871: 					$sql_left_join = "
1872: 						LEFT JOIN (
1873: 								SELECT
1874: 									`id_car`	
1875: 								FROM
1876: 									`car_users`
1877: 								WHERE
1878: 									`id_user` = '" . $_SESSION[UID] . "' " . $sql . "
```

### `archive_17012026_1259/taxi/models/api.php:1857`
```php
1847: 			}
1848: 			return $this->showError('404', 'error', 'wrong hash');
1849: 		}
1850: 
1851: 		public function selectCar($id_car = "", $id_user = "", $langs = array())
1852: 		{
1853: 			if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
1854: 			$sql = $sql_json = $sql_left_join = "";
1855: 			if (!empty($id_user))
1856: 			{
1857: 				if ($id_user == 'authorized') 
1858: 				{
1859: 					if (empty($id_car))
1860: 					{
1861: 						$sql = "";
1862: 					}
1863: 					elseif ($id_car == 'driven')
1864: 					{
1865: 						$sql = "AND `used` = '1'";
1866: 					}
1867: 					else
1868: 					{
1869: 						$sql = "AND `id_car` in (" . $id_car . ")";
1870: 					}
1871: 					$sql_left_join = "
1872: 						LEFT JOIN (
1873: 								SELECT
1874: 									`id_car`	
1875: 								FROM
1876: 									`car_users`
1877: 								WHERE
1878: 									`id_user` = '" . $_SESSION[UID] . "' " . $sql . "
1879: 								GROUP BY
1880: 
1881: 									`id_car`
1882: 							) cu USING (`id_car`)";
```

### `archive_17012026_1259/taxi/models/api.php:2045`
```php
2035: 					}
2036: 				}
2037: 				$out['car'][$d['c_id']] = $d;
2038: 			}
2039: 			if (empty($this->associativeArray)) $out['car'] =  array_values($out['car']);
2040: 
2041: 			return array(
2042: 				'code' 		=>	'200',
2043: 				'status' 	=>	'success',		
2044: 				'data' 		=>	$out,
2045: 				'auth_user' =>	array(
2046: 									'u_id' => $_SESSION[UID],
2047: 									'u_name' => $_SESSION['name'],
2048: 									'u_family' => $_SESSION['family'],
2049: 									'u_middle' => $_SESSION['middle'],
2050: 									'u_email' => $_SESSION['email'],
2051: 									'u_phone' => $_SESSION['phone'],
2052: 									'u_role' => $_SESSION['id_role'],
2053: 									'u_a_role' => $this->id_role,
2054: 									'u_check_state' => $_SESSION['id_verification_status'],
2055: 									'u_ban' => $_SESSION['user_ban_status'],
2056: 									'u_active' => $_SESSION['active'],
2057: 									'u_photo' => $_SESSION['photo_link'],
2058: 									'u_birthday' => $_SESSION['birthday_date'],
2059: 									'u_lang' => $_SESSION['id_lang'],
2060: 									'u_currency' => $_SESSION['currency'],
2061: 									'u_gps_software' => $_SESSION['id_navigation']
2062: 								)
2063: 			);
2064: 		}
2065: 
2066: 		public function controlCar($data = "", $id_car = "", $id_user = "", $langs = array(), $order_location = array(), $currency = array(), $country = array(),$region = array(),$city = array())
2067: 		{
2068: 			if (empty($_SESSION[UID])) {
2069: 				return $this->showError('404', 'error', 'unauthorized access');
2070: 			}	
```

### `archive_17012026_1259/taxi/models/api.php:2069`
```php
2059: 									'u_lang' => $_SESSION['id_lang'],
2060: 									'u_currency' => $_SESSION['currency'],
2061: 									'u_gps_software' => $_SESSION['id_navigation']
2062: 								)
2063: 			);
2064: 		}
2065: 
2066: 		public function controlCar($data = "", $id_car = "", $id_user = "", $langs = array(), $order_location = array(), $currency = array(), $country = array(),$region = array(),$city = array())
2067: 		{
2068: 			if (empty($_SESSION[UID])) {
2069: 				return $this->showError('404', 'error', 'unauthorized access');
2070: 			}	
2071: 			if (empty($data)) 
2072: 			{
2073: 				return $this->showError('404', 'error', 'empty data');
2074: 			}
2075: 
2076: 			$data = json_decode($data,true);
2077: 			
2078: 			if (empty($data) || !is_array($data)) 
2079: 			{
2080: 				return $this->showError('404', 'error', 'wrong data');
2081: 			}
2082: 
2083: 			$allowed_fields = array(							
2084: 									'cm_id'					=>		'id_car_model',
2085: 									'seats'					=>		'seats',
2086: 									'registration_plate'	=>		'license_plate',
2087: 									'color'					=>		'id_car_color',
2088: 									'photo'					=>		'photo_link',
2089: 									'details'				=>		'json',
2090: 									'cc_id'					=>		'id_car_class',
2091: 									'licenses'				=>		empty($id_car) ? '' : 'licenses'
2092: 			);
2093: 
2094: 			$forbidden_fields = array();
```

### `archive_17012026_1259/taxi/models/api.php:2847`
```php
2837: 				$q = query("COMMIT");
2838: 				if ($q === false) return array('error' => 'commit query failed');
2839: 			}
2840: 		
2841: 			return true;
2842: 		}
2843: 
2844: 		public function createOrder($data = array(), $langs = array(), $payment_method = array(), $payment_card = array(), $comment_items = array(), $order_service = array(), $contact = array(), $order_location = array(), $currency = array(), $unit_set = array(), $country = array(),$region = array(),$city = array(), $schedule = array(), $price_time_functions = array(), $stripe_seat_title_template = '', $stripe_request_duration = 0, $aggregators = array())
2845: 		{
2846: 			if (empty($_SESSION[UID])) {
2847: 				return $this->showError('404', 'error', 'unauthorized access');
2848: 			}
2849: 			if ($this->id_role != 1 && $this->id_role != 5 && $this->id_role != 4)
2850: 			{
2851: 				return $this->showError('404', 'error', 'wrong user role');
2852: 			}
2853: 			if (!empty($_SESSION['user_ban_status']['order']))
2854: 			{
2855: 				return $this->showError('404', 'error', 'user banned');
2856: 			}	
2857: 			
2858: 			if (empty($data)) 
2859: 			{
2860: 				return $this->showError('404', 'error', 'empty data');
2861: 			}
2862: 
2863: 			$data = json_decode($data,true);
2864: 			
2865: 			if (empty($data) || !is_array($data)) 
2866: 			{
2867: 				return $this->showError('404', 'error', 'wrong data');
2868: 			}
2869: 			$iso = array();
2870: 			foreach ($langs as $lang)
2871: 			{
2872: 				$iso[$lang['iso']] = array(
```

### `archive_17012026_1259/taxi/models/api.php:4478`
```php
4468: 			return array(
4469: 				'code' 		=>	'200',
4470: 				'status' 	=>	'success',
4471: 				'data' 		=>	$out
4472: 			);
4473: 		}
4474: 
4475: 		public function selectActiveOrder($fields = 0)
4476: 		{
4477: 			if (empty($_SESSION[UID])) {
4478: 				return $this->showError('404', 'error', 'unauthorized access');
4479: 			}
4480: 
4481: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
4482: 			{
4483: 				return $this->showError('404', 'error', 'wrong user role');
4484: 			}
4485: 
4486: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = $sql_where = '';
4487: 						
4488: 			$field_flag = array();
4489: 			if (!empty($fields))
4490: 			{
4491: 				$field_arr	= get_field_arr('activeOrder',$this->id_role);
4492: 				$bin_arr = get_bin_arr();
4493: 
4494: 				foreach(str_split($fields) as $index => $char)
4495: 				{
4496: 					$value = get_0_64($char);
4497: 					if (empty($value)) continue;
4498: 					foreach($bin_arr as $bin_i)
4499: 					{
4500: 						if ($value & $bin_i) 
4501: 						{
4502: 							if (isset($field_arr[$index][$bin_i]))
4503: 							{
```

### `archive_17012026_1259/taxi/models/api.php:4922`
```php
4912: 			if (empty($this->associativeArray)) $out['booking'] =  array_values($out['booking']);
4913: 
4914: 			$out['user'] = $this->getUsersForOrder(empty($field_flag['users_client']) ? array() : $users_client,empty($field_flag['stat_client']) ? array() : $stat_client,empty($field_flag['users_driver']) ? array() : $users_driver,empty($field_flag['stat_driver']) ? array() : $stat_driver );
4915: 			if (!empty($out['user']['error']))  return $this->showError('404', 'error', "user info: {$out['user']['error']}");
4916: 			if (empty($out['user'])) unset($out['user']);
4917: 
4918: 			return array(
4919: 				'code' 		=>	'200',
4920: 				'status' 	=>	'success',		
4921: 				'data' 		=>	$out,
4922: 				'auth_user' =>	array(
4923: 									'u_id' => $_SESSION[UID],
4924: 									'u_name' => $_SESSION['name'],
4925: 									'u_family' => $_SESSION['family'],
4926: 									'u_middle' => $_SESSION['middle'],
4927: 									'u_email' => $_SESSION['email'],
4928: 									'u_phone' => $_SESSION['phone'],
4929: 									'u_role' => $_SESSION['id_role'],
4930: 									'u_a_role' => $this->id_role,
4931: 									'u_check_state' => $_SESSION['id_verification_status'],
4932: 									'u_ban' => $_SESSION['user_ban_status'],
4933: 									'u_active' => $_SESSION['active'],
4934: 									'u_photo' => $_SESSION['photo_link'],
4935: 									'u_birthday' => $_SESSION['birthday_date'],
4936: 									'u_lang' => $_SESSION['id_lang'],
4937: 									'u_currency' => $_SESSION['currency'],
4938: 									'u_gps_software' => $_SESSION['id_navigation']
4939: 								)
4940: 			);
4941: 		}
4942: 
4943: 		public function selectProcessingOrder($fields = 0, $filter = NULL, $order_location = array())
4944: 		{
4945: 			if (empty($_SESSION[UID])) {
4946: 				return $this->showError('404', 'error', 'unauthorized access');
4947: 			}
```

### `archive_17012026_1259/taxi/models/api.php:4946`
```php
4936: 									'u_lang' => $_SESSION['id_lang'],
4937: 									'u_currency' => $_SESSION['currency'],
4938: 									'u_gps_software' => $_SESSION['id_navigation']
4939: 								)
4940: 			);
4941: 		}
4942: 
4943: 		public function selectProcessingOrder($fields = 0, $filter = NULL, $order_location = array())
4944: 		{
4945: 			if (empty($_SESSION[UID])) {
4946: 				return $this->showError('404', 'error', 'unauthorized access');
4947: 			}
4948: 			if ($this->id_role != 2 && $this->id_role != 4)
4949: 			{
4950: 				return $this->showError('404', 'error', 'wrong user role');
4951: 			}
4952: 
4953: 			$sql_order = $sql_drivers = $sql_order_driver = $sql_c_options = $sql_payment = $sql_left_join = $sql_where_order = $sql_where = '';
4954: 		
4955: 			$field_flag = array();
4956: 			if (!empty($fields))
4957: 			{
4958: 				$field_arr	= get_field_arr('processingOrder',$this->id_role);
4959: 				$bin_arr = get_bin_arr();
4960: 
4961: 				foreach(str_split($fields) as $index => $char)
4962: 				{
4963: 					$value = get_0_64($char);
4964: 					if (empty($value)) continue;
4965: 					foreach($bin_arr as $bin_i)
4966: 					{
4967: 						if ($value & $bin_i) 
4968: 						{
4969: 							if (isset($field_arr[$index][$bin_i]))
4970: 							{
4971: 								$field_flag[$field_arr[$index][$bin_i]] = true;
```

### `archive_17012026_1259/taxi/models/api.php:5582`
```php
5572: 			if (empty($this->associativeArray)) $out['booking'] =  array_values($out['booking']);
5573: 
5574: 			$out['user'] = $this->getUsersForOrder(empty($field_flag['users_client']) ? array() : $users_client,empty($field_flag['stat_client']) ? array() : $stat_client,empty($field_flag['users_driver']) ? array() : $users_driver,empty($field_flag['stat_driver']) ? array() : $stat_driver );
5575: 			if (!empty($out['user']['error']))  return $this->showError('404', 'error', "user info: {$out['user']['error']}");
5576: 			if (empty($out['user'])) unset($out['user']);
5577: 
5578: 			return array(
5579: 				'code' 		=>	'200',
5580: 				'status' 	=>	'success',		
5581: 				'data' 		=>	$out,
5582: 				'auth_user' =>	array(
5583: 									'u_id' => $_SESSION[UID],
5584: 									'u_name' => $_SESSION['name'],
5585: 									'u_family' => $_SESSION['family'],
5586: 									'u_middle' => $_SESSION['middle'],
5587: 									'u_email' => $_SESSION['email'],
5588: 									'u_phone' => $_SESSION['phone'],
5589: 									'u_role' => $_SESSION['id_role'],
5590: 									'u_a_role' => $this->id_role,
5591: 									'u_check_state' => $_SESSION['id_verification_status'],
5592: 									'u_ban' => $_SESSION['user_ban_status'],
5593: 									'u_active' => $_SESSION['active'],
5594: 									'u_photo' => $_SESSION['photo_link'],
5595: 									'u_birthday' => $_SESSION['birthday_date'],
5596: 									'u_lang' => $_SESSION['id_lang'],
5597: 									'u_currency' => $_SESSION['currency'],
5598: 									'u_gps_software' => $_SESSION['id_navigation']
5599: 								)
5600: 			);
5601: 		}
5602: 
5603: 		public function selectArchiveOrder($fields = 0)
5604: 		{
5605: 			if (empty($_SESSION[UID])) {
5606: 				return $this->showError('404', 'error', 'unauthorized access');
5607: 			}
```

### `archive_17012026_1259/taxi/models/api.php:5606`
```php
5596: 									'u_lang' => $_SESSION['id_lang'],
5597: 									'u_currency' => $_SESSION['currency'],
5598: 									'u_gps_software' => $_SESSION['id_navigation']
5599: 								)
5600: 			);
5601: 		}
5602: 
5603: 		public function selectArchiveOrder($fields = 0)
5604: 		{
5605: 			if (empty($_SESSION[UID])) {
5606: 				return $this->showError('404', 'error', 'unauthorized access');
5607: 			}
5608: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
5609: 			{
5610: 				return $this->showError('404', 'error', 'wrong user role');
5611: 			}
5612: 
5613: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = $sql_where = '';
5614: 						
5615: 			$field_flag = array();
5616: 			if (!empty($fields))
5617: 			{
5618: 				$field_arr	= get_field_arr('archiveOrder',$this->id_role);
5619: 				$bin_arr = get_bin_arr();
5620: 
5621: 				foreach(str_split($fields) as $index => $char)
5622: 				{
5623: 					$value = get_0_64($char);
5624: 					if (empty($value)) continue;
5625: 					foreach($bin_arr as $bin_i)
5626: 					{
5627: 						if ($value & $bin_i) 
5628: 						{
5629: 							if (isset($field_arr[$index][$bin_i]))
5630: 							{
5631: 								$field_flag[$field_arr[$index][$bin_i]] = true;
```

### `archive_17012026_1259/taxi/models/api.php:6159`
```php
6149: 			if (empty($this->associativeArray)) $out['booking'] =  array_values($out['booking']);
6150: 
6151: 			$out['user'] = $this->getUsersForOrder(empty($field_flag['users_client']) ? array() : $users_client,empty($field_flag['stat_client']) ? array() : $stat_client,empty($field_flag['users_driver']) ? array() : $users_driver,empty($field_flag['stat_driver']) ? array() : $stat_driver );
6152: 			if (!empty($out['user']['error']))  return $this->showError('404', 'error', "user info: {$out['user']['error']}");
6153: 			if (empty($out['user'])) unset($out['user']);
6154: 
6155: 			return array(
6156: 				'code' 		=>	'200',
6157: 				'status' 	=>	'success',		
6158: 				'data' 		=>	$out,
6159: 				'auth_user' =>	array(
6160: 									'u_id' => $_SESSION[UID],
6161: 									'u_name' => $_SESSION['name'],
6162: 									'u_family' => $_SESSION['family'],
6163: 									'u_middle' => $_SESSION['middle'],
6164: 									'u_email' => $_SESSION['email'],
6165: 									'u_phone' => $_SESSION['phone'],
6166: 									'u_role' => $_SESSION['id_role'],
6167: 									'u_a_role' => $this->id_role,
6168: 									'u_check_state' => $_SESSION['id_verification_status'],
6169: 									'u_ban' => $_SESSION['user_ban_status'],
6170: 									'u_active' => $_SESSION['active'],
6171: 									'u_photo' => $_SESSION['photo_link'],
6172: 									'u_birthday' => $_SESSION['birthday_date'],
6173: 									'u_lang' => $_SESSION['id_lang'],
6174: 									'u_currency' => $_SESSION['currency'],
6175: 									'u_gps_software' => $_SESSION['id_navigation']
6176: 								)
6177: 			);
6178: 		}
6179: 
6180: 		public function setDriver($data = '', $id_order = "", $id_user = "", $appointed_performer = 0, $code = NULL, $trips = "")
6181: 		{
6182: 			if (empty($_SESSION[UID])) {
6183: 				return $this->showError('404', 'error', 'unauthorized access');
6184: 			}
```

### `archive_17012026_1259/taxi/models/api.php:6183`
```php
6173: 									'u_lang' => $_SESSION['id_lang'],
6174: 									'u_currency' => $_SESSION['currency'],
6175: 									'u_gps_software' => $_SESSION['id_navigation']
6176: 								)
6177: 			);
6178: 		}
6179: 
6180: 		public function setDriver($data = '', $id_order = "", $id_user = "", $appointed_performer = 0, $code = NULL, $trips = "")
6181: 		{
6182: 			if (empty($_SESSION[UID])) {
6183: 				return $this->showError('404', 'error', 'unauthorized access');
6184: 			}
6185: 
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
6189: 			}
6190: 			
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
```

### `archive_17012026_1259/taxi/models/api.php:6218`
```php
6208: 
6209: 				$q = query($s);
6210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6211: 				
6212: 				$d = fetch_assoc($q);
6213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6214: 				if ($d['id_order_status'] != 1) return $this->showError('404', 'error', 'wrong booking state');
6215: 
6216: 				if ($d['client'] != $_SESSION[UID]) 
6217: 				{
6218: 					return $this->showError('404', 'error', $_SESSION[UID] . ' is not author');
6219: 				}
6220: 				if (empty($d['u_id'])) 
6221: 				{
6222: 					return $this->showError('404', 'error', $id_user . ' is not performer');
6223: 				}
6224: 				if (empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking not confirmed');
6225: 				if ($d['c_state'] != 1) 
6226: 				{
6227: 					return $this->showError('404', 'error', 'wrong booking driver state');
6228: 				}
6229: 
6230: 				$q = query("BEGIN");
6231: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
6232: 	
6233: 				$s = "UPDATE `order`,`order_driver`,(
6234: 							SELECT
6235: 
6236: 								'" . $id_order . "' as `id_order`,
6237: 								@c_c_count := COUNT(`id_order`) as current_car_count
6238: 							FROM
6239: 								`order_driver`
6240: 							WHERE
6241: 								`id_order` = '" . $id_order . "' AND 
6242: 								`id_order_driver_status` in (3,4,5,6)
6243: 
```

### `archive_17012026_1259/taxi/models/api.php:6316`
```php
6306: 						FROM `trip` 		
6307: 						WHERE	
6308: 							`id_trip` in (" . $trips . ") AND `driver` = '" . $id_user . "'
6309: 						";
6310: 
6311: 					$q = query($s);
6312: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
6313: 
6314: 					$d = fetch_assoc($q);
6315: 					$trips = explode(',', $trips);
6316: 					if ($d['trips_count'] != count($trips)) return $this->showError('404', 'error', 'driver is not trip author');
6317: 					
6318: 					$s = array();
6319: 					foreach ($trips as $id_trip)
6320: 					{
6321: 						$s[] = "('" . $id_order . "', '" . $id_trip . "', now(), now())";
6322: 					}
6323: 					$s = "INSERT INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`, `select_trip_datetime`) VALUES " . implode(",", $s) . "ON DUPLICATE KEY UPDATE `select_trip_datetime` = IF(`select_trip_datetime` = 0,now(),`select_trip_datetime`)";
6324: 
6325: 					$q = query($s);
6326: 					if ($q === false) return $this->showError('404', 'error', 'insert in database failed');
6327: 				}		
6328: 
6329: 				$q = query("COMMIT");
6330: 				if ($q === false) return $this->showError('404', 'error', 'commit query failed');
6331: 
6332: 				if ($rows === 1 && $d_u['id'] == 2) 
6333: 				{
6334: 					return $this->showError('404', 'error', 'only booking state modified');	
6335: 				}
6336: 				
6337: 				$out = array('current_cars_count' => (int)$d_u['@c_c_count']+1,'b_cars_count' => $d_u['@c_count']);
6338: 				if ($d_u['@id'] == 2) $out['b_state'] = '1->2';
6339: 			
6340: 			}
6341: 			else
```

### `archive_17012026_1259/taxi/models/api.php:6439`
```php
6429: 				$q = query($s);
6430: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6431: 				
6432: 				$d = fetch_assoc($q);
6433: 				$d['car_count'] = (int)$d['car_count'];  
6434: 				$d['current_car_count'] = (int)$d['current_car_count'];
6435: 				$d['appointed_performer_count'] = (int)$d['appointed_performer_count'];
6436: 				$d['max_index'] = (int)$d['max_index'];
6437: 
6438: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6439: 				if ($d['client'] == $_SESSION[UID]) return $this->showError('404', 'error', 'driver is author');
6440: 				if ((int)$d['attempt'] >= $this->constant['driver_code_attempt_count_max'])
6441: 				{
6442: 					return $this->showError('404', 'error', 'maximum attempts exceeded');
6443: 				}
6444: 				
6445: 				if (isset($code)) {
6446: 					if (strlen($d['code']) == 0) 
6447: 					{
6448: 						return $this->showError('404', 'error', 'booking with empty driver code');
6449: 					}
6450: 					if ($d['code'] == $this->prepareForDriverCode($code))
6451: 					{
6452: 						$appointed_performer = 1;
6453: 					}
6454: 					else
6455: 					{
6456: 						$s = "INSERT INTO `order_driver_attempt`
6457: 							SET 
6458: 								`id_order` = '" . $id_order . "',
6459: 								`id_user` = '" . $_SESSION[UID]  . "',
6460: 								`datetime` = now()
6461: 							";
6462: 
6463: 						$q = query($s);
6464: 						if ($q === false) return $this->showError('404', 'error', 'database attempt insert failed');
```

### `archive_17012026_1259/taxi/models/api.php:6743`
```php
6733: 						FROM `trip` 		
6734: 						WHERE	
6735: 							`id_trip` in (" . $trips . ") AND `driver` = '" . $_SESSION[UID] . "'
6736: 						";
6737: 
6738: 					$q = query($s);
6739: 					if ($q === false) return $this->showError('404', 'error', 'select failed');
6740: 
6741: 					$d = fetch_assoc($q);
6742: 					$trips = explode(',', $trips);
6743: 					if ($d['trips_count'] != count($trips)) return $this->showError('404', 'error', 'driver is not trip author');
6744: 					
6745: 					$s = array();
6746: 					
6747: 					if ($id_order_driver_status == '1')
6748: 					{
6749: 						foreach ($trips as $id_trip)
6750: 						{
6751: 							$s[] = "('" . $id_order . "', '" . $id_trip . "', now())";
6752: 						}
6753: 						$s = "INSERT IGNORE INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`) VALUES " . implode(",", $s);
6754: 					}
6755: 					elseif ($id_order_driver_status == '3')
6756: 					{
6757: 						foreach ($trips as $id_trip)
6758: 						{
6759: 							$s[] = "('" . $id_order . "', '" . $id_trip . "', now(), now())";
6760: 						}
6761: 						$s = "INSERT INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`, `select_trip_datetime`) VALUES " . implode(",", $s) . "ON DUPLICATE KEY UPDATE `select_trip_datetime` = IF(`select_trip_datetime` = 0,now(),`select_trip_datetime`)";
6762: 					}
6763: 					
6764: 					$q = query($s);
6765: 					if ($q === false) return $this->showError('404', 'error', 'insert in database failed');
6766: 				}				
6767: 
6768: 				$s = "UPDATE `order`
```

### `archive_17012026_1259/taxi/models/api.php:6802`
```php
6792: 			return array(
6793: 				'code' 		=>	'200',
6794: 				'status' 	=>	'success',
6795: 				'data' 		=>	$out
6796: 			);	
6797: 		}
6798: 
6799: 		public function setCarIsArrived($id_order = "")
6800: 		{
6801: 			if (empty($_SESSION[UID])) {
6802: 				return $this->showError('404', 'error', 'unauthorized access');
6803: 			}
6804: 			if ($this->id_role != 2)
6805: 			{
6806: 				return $this->showError('404', 'error', 'wrong user role');
6807: 			}
6808: 
6809: 			$s = "SELECT
6810: 					`order`.`id_order`,
6811: 					`order`.`id_order_status`,
6812: 					od.`id_user`,
6813: 					od.`id_order_driver_status`
6814: 				FROM `order` 
6815: 				LEFT JOIN (
6816: 						SELECT
6817: 							`id_order`,
6818: 							`id_user`,
6819: 							`id_order_driver_status`
6820: 						FROM
6821: 							`order_driver`
6822: 						WHERE
6823: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
6824: 					) od USING (`id_order`)				
6825: 				WHERE	
6826: 					`order`.`id_order` = '" . $id_order . "'
6827: 				LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:6899`
```php
6889: 
6890: 			return array(
6891: 				'code' 		=>	'200',
6892: 				'status' 	=>	'success'
6893: 			);
6894: 		}
6895: 
6896: 		public function completeOrder($id_order = "")
6897: 		{
6898: 			if (empty($_SESSION[UID])) {
6899: 				return $this->showError('404', 'error', 'unauthorized access');
6900: 			}
6901: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6902: 			{
6903: 				return $this->showError('404', 'error', 'wrong user role');
6904: 			}
6905: 
6906: 			if ($this->id_role == 1 || $this->id_role == 5)
6907: 			{
6908: 				$s = "SELECT
6909: 						`id_order`,
6910: 						`client`,
6911: 						`id_order_status`
6912: 					FROM `order` 		
6913: 					WHERE	
6914: 						`id_order` = '" . $id_order . "'
6915: 					LIMIT 1
6916: 					";
6917: 
6918: 				$q = query($s);
6919: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6920: 
6921: 				$d = fetch_assoc($q);
6922: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6923: 				if ($d['id_order_status'] != 2) return $this->showError('404', 'error', 'wrong booking state');
6924: 				if ($d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:6924`
```php
6914: 						`id_order` = '" . $id_order . "'
6915: 					LIMIT 1
6916: 					";
6917: 
6918: 				$q = query($s);
6919: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6920: 
6921: 				$d = fetch_assoc($q);
6922: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6923: 				if ($d['id_order_status'] != 2) return $this->showError('404', 'error', 'wrong booking state');
6924: 				if ($d['client'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user is not author');
6925: 
6926: 				$s = "UPDATE `order`
6927: 					SET 
6928: 						`id_order_status` = '4',
6929: 						`complete_datetime` = now(),
6930: 						`last_edit_datetime` = now(),
6931: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
6932: 					WHERE
6933: 						`id_order` = '" . $id_order . "' AND `id_order_status` = '2'
6934: 					";
6935: 
6936: 				query($s);
6937: 		
6938: 				global $link;
6939: 				if (mysqli_affected_rows($link) === -1) 
6940: 				{
6941: 					return $this->showError('404', 'error', 'database update failed');
6942: 				}
6943: 				elseif (mysqli_affected_rows($link) === 0) 
6944: 				{
6945: 
6946: 					return $this->showError('404', 'error', 'modified data not found');
6947: 				}
6948: 			}
6949: 			elseif ($this->id_role == 2)
```

### `archive_17012026_1259/taxi/models/api.php:7053`
```php
7043: 
7044: 			return array(
7045: 				'code' 		=>	'200',
7046: 				'status' 	=>	'success'
7047: 			);	
7048: 		}
7049: 
7050: 		public function cancelOrder($id_order = "", $forced = 0, $cancel_reason = "", $cancel_states = "", $site_cancel_states = array())
7051: 		{
7052: 			if (empty($_SESSION[UID])) {
7053: 				return $this->showError('404', 'error', 'unauthorized access');
7054: 			}
7055: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5 && $this->id_role != 4)
7056: 			{
7057: 				return $this->showError('404', 'error', 'wrong user role');
7058: 			}
7059: 			if (!empty($cancel_states))
7060: 			{
7061: 				$s_cancel_states = array();
7062: 				$forbidden_cancel_states = array();
7063: 				foreach (explode(',', $cancel_states) as $c_s)
7064: 				{
7065: 					$c_s = trim($c_s);
7066: 					if (!empty($site_cancel_states[$c_s]['user_roles']) && in_array($this->id_role,$site_cancel_states[$c_s]['user_roles']))
7067: 					{
7068: 						$s_cancel_states[] = "('" . $id_order . "', '" . $c_s . "', '" . $_SESSION[UID] . "')";
7069: 					}
7070: 					else
7071: 					{
7072: 						$forbidden_cancel_states[] = $c_s;
7073: 					}
7074: 				}
7075: 				if (!empty($forbidden_cancel_states))
7076: 				{
7077: 					return $this->showError('404', 'error', 'wrong booking_cancel_states: ' . implode(",", $forbidden_cancel_states));
7078: 				}
```

### `archive_17012026_1259/taxi/models/api.php:7108`
```php
7098: 
7099: 				$d = fetch_assoc($q);
7100: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7101: 				if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5 
7102: 					&& $d['id_order_status'] != 6) 
7103: 				{
7104: 					return $this->showError('404', 'error', 'wrong booking state');
7105: 				}
7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID]) 
7107: 				{
7108: 					return $this->showError('404', 'error', 'user is not author');
7109: 				}
7110: 
7111: 				if (DEFAULT_PROFILE == 'stadium')
7112: 				{
7113: 					if ($d['pay_datetime'] !== '0000-00-00 00:00:00') return $this->showError('404', 'error', 'order has already been paid');
7114: 					$d['options'] = json_decode($d['options'],true);
7115: 					$d_options = $d['options'];
7116: 					if (empty($d['options']['tickets']['payment'])) $d['options']['tickets']['payment'] = '';
7117: 					if ($d['options']['tickets']['payment'] == 'succeeded')
7118: 					{
7119: 						return $this->showError('404', 'error', 'order with succeeded status');
7120: 					}
7121: 
7122: 					if ($d['options']['tickets']['payment'] !== 'failed')
7123: 					{
7124: 						$api_use = true;
7125: 						$order = $id_order;
7126: 						$status = 'failed';
7127: 						$cancel_status = cancel_stripe_link($id_order);
7128: 						if (!empty($cancel_status['error'])) 
7129: 						{
7130: 							if (empty($cancel_status['error'][1]))
7131: 							{
7132: 								if (empty($forced)) return $this->showError('404', 'error', $cancel_status['error']);
7133: 							}
```

### `archive_17012026_1259/taxi/models/api.php:7399`
```php
7389: 
7390: 			return array(
7391: 				'code' 		=>	'200',
7392: 				'status' 	=>	'success'
7393: 			);
7394: 		}
7395: 
7396: 		public function rateOrder($id_order = "", $rating = "")
7397: 		{
7398: 			if (empty($_SESSION[UID])) {
7399: 				return $this->showError('404', 'error', 'unauthorized access');
7400: 			}
7401: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
7402: 			{
7403: 				return $this->showError('404', 'error', 'wrong user role');
7404: 			}
7405: 
7406: 			if ($this->id_role == 1 || $this->id_role == 5)
7407: 			{
7408: 				$s = "SELECT
7409: 						`id_order`,
7410: 						`client`,
7411: 						`id_order_status`,
7412: 						`rating`
7413: 					FROM `order` 		
7414: 					WHERE	
7415: 						`id_order` = '" . $id_order . "'
7416: 					LIMIT 1
7417: 					";
7418: 
7419: 				$q = query($s);
7420: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7421: 
7422: 				$d = fetch_assoc($q);
7423: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7424: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
```

### `archive_17012026_1259/taxi/models/api.php:7427`
```php
7417: 					";
7418: 
7419: 				$q = query($s);
7420: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
7421: 
7422: 				$d = fetch_assoc($q);
7423: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
7424: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
7425: 				if ($d['client'] != $_SESSION[UID]) 
7426: 				{
7427: 					return $this->showError('404', 'error', 'user is not author');
7428: 				}
7429: 				if ($d['rating'] !== NULL) return $this->showError('404', 'error', 'booking already rated');
7430: 
7431: 				$s = "UPDATE `order`
7432: 					SET 
7433: 						`rating` = '" . $rating  . "',
7434: 						`last_edit_datetime` = now(),
7435: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
7436: 					WHERE
7437: 						`id_order` = '" . $id_order . "' AND `rating` IS NULL
7438: 					";
7439: 
7440: 				query($s);
7441: 		
7442: 				global $link;
7443: 				if (mysqli_affected_rows($link) === -1)
7444: 				{
7445: 					return $this->showError('404', 'error', 'database update failed');
7446: 				}
7447: 				elseif (mysqli_affected_rows($link) === 0)
7448: 				{
7449: 
7450: 					return $this->showError('404', 'error', 'modified data not found');
7451: 				}
7452: 			}
```

### `archive_17012026_1259/taxi/models/api.php:7529`
```php
7519: 
7520: 			return array(
7521: 				'code' 		=>	'200',
7522: 				'status' 	=>	'success'
7523: 			);
7524: 		}
7525: 
7526: 		public function selectOrder($id_order = "", $fields = 0)
7527: 		{
7528: 			if (empty($_SESSION[UID])) {
7529: 				return $this->showError('404', 'error', 'unauthorized access');
7530: 			}
7531: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
7532: 			{
7533: 				return $this->showError('404', 'error', 'wrong user role');
7534: 			}			
7535: 
7536: 			$sql_order = $sql_order_driver = $sql_c_options = $sql_left_join = '';
7537: 			if (empty($id_order))
7538: 			{
7539: 				$sql_where = '1=1';
7540: 			}
7541: 			else
7542: 			{
7543: 				$sql_where = "`order`.`id_order` in (" . $id_order . ")";
7544: 			}
7545: 
7546: 			$field_flag = array();
7547: 			if (!empty($fields))
7548: 			{
7549: 				$field_arr	= get_field_arr('order',$this->id_role);
7550: 				$bin_arr = get_bin_arr();
7551: 
7552: 				foreach(str_split($fields) as $index => $char)
7553: 				{
7554: 					$value = get_0_64($char);
```

### `archive_17012026_1259/taxi/models/api.php:8136`
```php
8126: 			if (empty($this->associativeArray)) $out['booking'] =  array_values($out['booking']);
8127: 
8128: 			$out['user'] = $this->getUsersForOrder(empty($field_flag['users_client']) ? array() : $users_client,empty($field_flag['stat_client']) ? array() : $stat_client,empty($field_flag['users_driver']) ? array() : $users_driver,empty($field_flag['stat_driver']) ? array() : $stat_driver );
8129: 			if (!empty($out['user']['error']))  return $this->showError('404', 'error', "user info: {$out['user']['error']}");
8130: 			if (empty($out['user'])) unset($out['user']);
8131: 
8132: 			return array(
8133: 				'code' 		=>	'200',
8134: 				'status' 	=>	'success',		
8135: 				'data' 		=>	$out,
8136: 				'auth_user' =>	array(
8137: 									'u_id' => $_SESSION[UID],
8138: 									'u_name' => $_SESSION['name'],
8139: 									'u_family' => $_SESSION['family'],
8140: 									'u_middle' => $_SESSION['middle'],
8141: 									'u_email' => $_SESSION['email'],
8142: 									'u_phone' => $_SESSION['phone'],
8143: 									'u_role' => $_SESSION['id_role'],
8144: 									'u_a_role' => $this->id_role,
8145: 
8146: 									'u_check_state' => $_SESSION['id_verification_status'],
8147: 									'u_ban' => $_SESSION['user_ban_status'],
8148: 									'u_active' => $_SESSION['active'],
8149: 									'u_photo' => $_SESSION['photo_link'],
8150: 									'u_birthday' => $_SESSION['birthday_date'],
8151: 									'u_lang' => $_SESSION['id_lang'],
8152: 									'u_currency' => $_SESSION['currency'],
8153: 									'u_gps_software' => $_SESSION['id_navigation']
8154: 								)
8155: 			);
8156: 		}
8157: 
8158: 		public function setLocation($lat = "", $lng = "")
8159: 		{
8160: 			if (empty($_SESSION[UID])) {
8161: 				return $this->showError('404', 'error', 'unauthorized access');
```

### `archive_17012026_1259/taxi/models/api.php:8161`
```php
8151: 									'u_lang' => $_SESSION['id_lang'],
8152: 									'u_currency' => $_SESSION['currency'],
8153: 									'u_gps_software' => $_SESSION['id_navigation']
8154: 								)
8155: 			);
8156: 		}
8157: 
8158: 		public function setLocation($lat = "", $lng = "")
8159: 		{
8160: 			if (empty($_SESSION[UID])) {
8161: 				return $this->showError('404', 'error', 'unauthorized access');
8162: 			}
8163: 
8164: 			$s = "REPLACE INTO `users_location`
8165: 				SET 
8166: 					`id_user` = '" . $_SESSION[UID]  . "',
8167: 					`lat` = '" . $lat  . "',
8168: 					`lng` = '" . $lng  . "',
8169: 					`datetime` = now()
8170: 				";
8171: 
8172: 			$q = query($s);
8173: 			if ($q === false) return $this->showError('404', 'error', 'database replace failed');
8174: 
8175: 			return array(
8176: 				'code' 		=>	'200',
8177: 				'status' 	=>	'success'
8178: 			);
8179: 		}
8180: 		
8181: 		public function confirmOrder($id_order = "", $estimated_waiting_time = NULL)
8182: 		{	
8183: 			if (empty($_SESSION[UID])) {
8184: 				return $this->showError('404', 'error', 'unauthorized access');
8185: 			}
8186: 			if ($this->id_role != 1 && $this->id_role != 5)
```

### `archive_17012026_1259/taxi/models/api.php:8184`
```php
8174: 
8175: 			return array(
8176: 				'code' 		=>	'200',
8177: 				'status' 	=>	'success'
8178: 			);
8179: 		}
8180: 		
8181: 		public function confirmOrder($id_order = "", $estimated_waiting_time = NULL)
8182: 		{	
8183: 			if (empty($_SESSION[UID])) {
8184: 				return $this->showError('404', 'error', 'unauthorized access');
8185: 			}
8186: 			if ($this->id_role != 1 && $this->id_role != 5)
8187: 			{
8188: 				return $this->showError('404', 'error', 'wrong user role');
8189: 			}
8190: 
8191: 			$s = "SELECT
8192: 					`id_order`,
8193: 					`client`,
8194: 					`id_order_status`,
8195: 					`is_confirmed`
8196: 				FROM `order` 		
8197: 				WHERE	
8198: 					`id_order` = '" . $id_order . "'
8199: 				LIMIT 1
8200: 				";
8201: 
8202: 			$q = query($s);
8203: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8204: 
8205: 			$d = fetch_assoc($q);
8206: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8207: 			if ($d['client'] != $_SESSION[UID]) 
8208: 			{
8209: 				return $this->showError('404', 'error', 'user is not author');
```

### `archive_17012026_1259/taxi/models/api.php:8209`
```php
8199: 				LIMIT 1
8200: 				";
8201: 
8202: 			$q = query($s);
8203: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8204: 
8205: 			$d = fetch_assoc($q);
8206: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8207: 			if ($d['client'] != $_SESSION[UID]) 
8208: 			{
8209: 				return $this->showError('404', 'error', 'user is not author');
8210: 
8211: 			}
8212: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 2 && $d['id_order_status'] != 5
8213: 				&& $d['id_order_status'] != 6)
8214: 			{
8215: 				return $this->showError('404', 'error', 'wrong booking state');
8216: 			}		
8217: 			if (!empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking already confirmed');
8218: 
8219: 			$s = "UPDATE `order`
8220: 				SET 
8221: 					`is_confirmed` = '1',
8222: 					`confirm_datetime` = now() " . ($estimated_waiting_time !== NULL ? ",
8223: 					`estimated_waiting_datetime` = IF(`datetime_start_plan` = 0,`create_datetime`,`datetime_start_plan`) + INTERVAL '" . $estimated_waiting_time . "' SECOND" : '') . ",
8224: 					`last_edit_datetime` = now(),
8225: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
8226: 				WHERE
8227: 					`id_order` = '" . $id_order . "' AND `is_confirmed` = '0'
8228: 				";
8229: 
8230: 			query($s);
8231: 	
8232: 			global $link;
8233: 			if (mysqli_affected_rows($link) === -1) 
8234: 			{
```

### `archive_17012026_1259/taxi/models/api.php:8251`
```php
8241: 
8242: 			return array(
8243: 				'code' 		=>	'200',
8244: 				'status' 	=>	'success'
8245: 			);
8246: 		}
8247: 		
8248: 		public function startOrder($id_order = "")
8249: 		{
8250: 			if (empty($_SESSION[UID])) {
8251: 				return $this->showError('404', 'error', 'unauthorized access');
8252: 			}
8253: 			if ($this->id_role != 2)
8254: 			{
8255: 				return $this->showError('404', 'error', 'wrong user role');
8256: 			}
8257: 
8258: 			$s = "SELECT
8259: 					`order`.`id_order`,
8260: 					`order`.`id_order_status`,
8261: 					od.`id_user`,
8262: 					od.`id_order_driver_status`
8263: 				FROM `order` 
8264: 				LEFT JOIN (
8265: 						SELECT
8266: 							`id_order`,
8267: 							`id_user`,
8268: 							`id_order_driver_status`
8269: 						FROM
8270: 							`order_driver`
8271: 						WHERE
8272: 							`id_order` = '" . $id_order . "' AND `id_user` = '" . $_SESSION[UID] . "'
8273: 					) od USING (`id_order`)				
8274: 				WHERE	
8275: 					`order`.`id_order` = '" . $id_order . "'
8276: 				LIMIT 1
```

### `archive_17012026_1259/taxi/models/api.php:8344`
```php
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
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
```

### `archive_17012026_1259/taxi/models/api.php:8346`
```php
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
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
```

### `archive_17012026_1259/taxi/models/api.php:8352`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:8353`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:8362`
```php
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
```

### `archive_17012026_1259/taxi/models/api.php:8365`
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
8390: 									'u_id' => $_SESSION[UID],
```

### `archive_17012026_1259/taxi/models/api.php:8389`
```php
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
8411: 		private $constant = array(
8412: 			'waiting_interval_or'					=>	array(
8413: 														'default'	=>	180,
8414: 														'type'		=>	'non-zero unsigned integer'
```

### `archive_17012026_1259/taxi/models/api.php:8425`
```php
8415: 													),
8416: 			'waiting_interval'						=>	array(
8417: 														'default'	=>	180,
8418: 														'type'		=>	'unsigned integer'
8419: 													),
8420: 			'waiting_interval_add'					=>	array(
8421: 														'default'	=>	180,
8422: 														'type'		=>	'non-zero unsigned integer'
8423: 													),
8424: 
8425: 			'token_interval_for_auth_hash'			=>	array(
8426: 														'default'	=>	10,
8427: 														'type'		=>	'unsigned integer'
8428: 													),
8429: 
8430: 			'limit_row_count'						=>	array(
8431: 														'default'	=>	5,
8432: 														'type'		=>	'non-zero unsigned integer'
8433: 													),
8434: 			'limit_row_count_max'					=>	array(
8435: 														'default'	=>	30,
8436: 														'type'		=>	'non-zero unsigned integer'
8437: 													),
8438: 
8439: 			'average_speed'							=>	array(
8440: 														'default'	=>	60,
8441: 														'type'		=>	'non-zero unsigned integer'
8442: 													),
8443: 
8444: 			'rating_max_b_c'						=>	array(
8445: 														'default'	=>	5,
8446: 														'type'		=>	'non-zero unsigned integer'
8447: 													),
8448: 			'rating_max_b_d'						=>	array(
8449: 														'default'	=>	5,
8450: 														'type'		=>	'non-zero unsigned integer'
```

### `archive_17012026_1259/taxi/models/api.php:8650`
```php
8640: 														'type'		=>	'json multidimensional associative list'
8641: 													),		
8642: 			't_options_edit_list_readonly'			=>	array(
8643: 														'default'	=>	false,
8644: 														'type'		=>	'boolean'
8645: 													),
8646: 			'start_datetime_interval'				=>	array(
8647: 														'default'	=>	86400,
8648: 														'type'		=>	'unsigned integer'
8649: 													),
8650: 			'auth_code_interval'			=>	array(
8651: 														'default'	=>	600,
8652: 														'type'		=>	'unsigned integer'
8653: 													),
8654: 			'ticket_booking_duration'			=>	array(
8655: 														'default'	=>	3600,
8656: 														'type'		=>	'unsigned integer'
8657: 													),		
8658: 			'query_extended_statements'			=>	array(
8659: 														'default'	=>	false,
8660: 														'type'		=>	'boolean'
8661: 													),
8662: 			'email_date_format'			=>	array(
8663: 														'default'	=>	'Y-m-d H:i:s',
8664: 														'type'		=>	'date format'
8665: 													),
8666: 			'order_payment'				=>	array(
8667: 														'default'	=>	0,
8668: 														'type'		=>	'unsigned integer'
8669: 													),								
8670: 			'session_token_duration'	=>	array(
8671: 														'default'	=>	86400,
8672: 														'type'		=>	'unsigned integer'
8673: 													),
8674: 			'placing_order_duration'	=>	array(
8675: 														'default'	=>	300,
```

### `archive_17012026_1259/taxi/models/api.php:8742`
```php
8732: 		private $car_class = array();
8733: 
8734: 		public function editWaitingTime($id_order = "", $previous = NULL, $additional = NULL)
8735: 		{
8736: 			if ($previous === NULL) return $this->showError('404', 'error', 'empty previous');
8737: 			if ($additional === NULL) $additional = $this->constant['waiting_interval_add'];
8738: 
8739: 			$additional = (int)$additional;
8740: 			if (empty($additional)) return $this->showError('404', 'error', 'empty additional');
8741: 			if (empty($_SESSION[UID])) {
8742: 				return $this->showError('404', 'error', 'unauthorized access');
8743: 			}
8744: 			if ($this->id_role != 1 && $this->id_role != 5)
8745: 			{
8746: 				return $this->showError('404', 'error', 'wrong user role');
8747: 			}
8748: 
8749: 			$q = query("BEGIN");
8750: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
8751: 
8752: 			$s = "SELECT
8753: 					`order`.`id_order`,
8754: 					`order`.`client`,
8755: 					`order`.`id_order_status`,
8756: 					TIMESTAMPDIFF(SECOND,IF(`order`.`datetime_start_plan` = 0,`order`.`create_datetime`,`order`.`datetime_start_plan`),`order`.`max_waiting_datetime`) as max_waiting_time,
8757: 					MAX(`order_waiting_time`.`index`) as max_index
8758: 				FROM `order` 
8759: 				LEFT JOIN `order_waiting_time` USING (`id_order`)		
8760: 				WHERE	
8761: 					`order`.`id_order` = '" . $id_order . "'
8762: 				LIMIT 1
8763: 				FOR UPDATE
8764: 				";
8765: 
8766: 			$q = query($s);
8767: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

### `archive_17012026_1259/taxi/models/api.php:8773`
```php
8763: 				FOR UPDATE
8764: 				";
8765: 
8766: 			$q = query($s);
8767: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
8768: 
8769: 			$d = fetch_assoc($q);
8770: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
8771: 			if ($d['client'] != $_SESSION[UID]) 
8772: 			{
8773: 				return $this->showError('404', 'error', 'user is not author');
8774: 			}
8775: 			if ($d['id_order_status'] != 1 && $d['id_order_status'] != 5 && $d['id_order_status'] != 6) 
8776: 			{
8777: 				return $this->showError('404', 'error', 'wrong booking state');
8778: 			}	
8779: 
8780: 			$d['max_waiting_time'] = (int)$d['max_waiting_time'];
8781: 			$previous = (int)$previous;
8782: 			if ($d['max_waiting_time'] !== $previous)
8783: 			{
8784: 				return $this->showError('404', 'error', 'wrong previous');
8785: 			}
8786: 			$d['max_index'] = (int)$d['max_index'];
8787: 
8788: 			$s = "INSERT INTO `order_waiting_time`
8789: 				SET 
8790: 					`id_order` = '" . $id_order . "',
8791: 					`interval` = '" . $additional . "',
8792: 					`datetime` = now(),
8793: 					`index` = '" . ($d['max_index'] + 1) . "'
8794: 				";
8795: 
8796: 			$q = query($s);
8797: 			if ($q === false) return $this->showError('404', 'error', 'database insert failed');
8798: 			
```

### `archive_17012026_1259/taxi/models/api.php:8970`
```php
8960: 			}
8961: 
8962: 			$this->limit_row_count = $this->constant['limit_row_count'];
8963: 
8964: 			$this->car_class = $car_class;
8965: 		}
8966: 		
8967: 		public function setCarUsed($id_car = "")
8968: 		{
8969: 			if (empty($_SESSION[UID])) {
8970: 				return $this->showError('404', 'error', 'unauthorized access');
8971: 			}
8972: 			if ($this->id_role != 2)
8973: 			{
8974: 				return $this->showError('404', 'error', 'wrong user role');
8975: 			}
8976: 			if ($_SESSION['id_verification_status'] != 2)
8977: 			{
8978: 				return $this->showError('404', 'error', 'wrong user check state');
8979: 			}
8980: 
8981: 			$q = query("BEGIN");
8982: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
8983: 			
8984: 			$s = "SELECT
8985: 					`car`.`id_car`,
8986: 					GROUP_CONCAT(IF(`car_users`.`id_user` = '" . $_SESSION[UID] . "',`car_users`.`id_user`,NULL) SEPARATOR ',') as u_id,
8987: 					GROUP_CONCAT(IF(`car_users`.`used` = '1',`car_users`.`id_user`,NULL) SEPARATOR ',') as u_d_id
8988: 				FROM `car`
8989: 				LEFT JOIN `car_users` USING (`id_car`)
8990: 				WHERE
8991: 					`car`.`id_car` = '" . $id_car . "'
8992: 				GROUP BY
8993: 					`car_users`.`id_car`
8994: 				";
8995: 
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
9093: 			{
9094: 				return $this->showError('404', 'error', 'wrong login');
9095: 			}
9096: 			elseif (!empty($d['deleted']))
9097: 			{
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
9115: 			query($s);
9116: 		
9117: 			if ($q === false) return $this->showError('404', 'error', 'database update failed');
9118: 
9119: 			if (!empty($d['email']))
```

### `archive_17012026_1259/taxi/models/api.php:9101`
```php
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
9115: 			query($s);
9116: 		
9117: 			if ($q === false) return $this->showError('404', 'error', 'database update failed');
9118: 
9119: 			if (!empty($d['email']))
9120: 			{
9121: 			
9122: 				$dataForEmail = array(
9123: 					'{$u_id}' 			=> $d['id_user'],
9124: 					'{$password}' 		=> $pwd,			
9125: 					'{$locationPath}' 	=> url('',CONFIG_URL),
9126: 					'{$u_name}' 		=> $d['name'],
```

### `archive_17012026_1259/taxi/models/api.php:9149`
```php
9139: 			return array(
9140: 				'code' 		=>	'200',
9141: 				'status' 	=>	'success'
9142: 			);
9143: 			
9144: 		}
9145: 
9146: 		public function newpass($password = "", $new_password = "")
9147: 		{
9148: 			if (empty($_SESSION[UID])) {
9149: 				return $this->showError('404', 'error', 'unauthorized access');
9150: 			}
9151: 
9152: 			$s = "SELECT 
9153: 					`id_user`
9154: 				FROM `users`
9155: 				WHERE 
9156: 					`id_user` = '" . $_SESSION[UID] . "' AND `pwd` = '" . md5(md5($password)) . "'
9157: 				LIMIT 1
9158: 				";
9159: 
9160: 			$q = query($s);
9161: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
9162: 			$d = fetch_assoc($q);
9163: 
9164: 			if (empty($d['id_user']))
9165: 			{
9166: 				return $this->showError('404', 'error', 'wrong password');
9167: 			}
9168: 
9169: 			$s = "UPDATE `users`
9170: 				SET 
9171: 					`pwd` = '" . md5(md5($new_password)) . "'
9172: 				WHERE
9173: 					`id_user` = '" . $_SESSION[UID] . "'
9174: 				";
```

### `archive_17012026_1259/taxi/models/api.php:9253`
```php
9243: 			return array(
9244: 				'code' 		=>	'200',
9245: 				'status' 	=>	'success'
9246: 
9247: 			);
9248: 		}
9249: 		
9250: 		public function editOrder($data = "", $id_order = "")
9251: 		{
9252: 			if (empty($_SESSION[UID])) {
9253: 				return $this->showError('404', 'error', 'unauthorized access');
9254: 			}
9255: 
9256: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
9257: 			{
9258: 				return $this->showError('404', 'error', 'wrong user role');
9259: 			}
9260: 
9261: 			if (empty($data)) 
9262: 			{
9263: 				return $this->showError('404', 'error', 'empty data');
9264: 			}
9265: 
9266: 			$data = json_decode($data,true);
9267: 			
9268: 			if (empty($data) || !is_array($data)) 
9269: 			{
9270: 				return $this->showError('404', 'error', 'wrong data');
9271: 			}
9272: 
9273: 			$allowed_fields = array(
9274: 									'b_start_address'			=>		'from',
9275: 									'b_start_latitude'			=>		'from_lat',
9276: 									'b_start_longitude'			=>		'from_lng',
9277: 									'b_destination_address'		=>		'to',
9278: 									'b_destination_latitude'	=>		'to_lat',
```

### `archive_17012026_1259/taxi/models/api.php:9346`
```php
9336: 					FOR UPDATE
9337: 					";
9338: 
9339: 				$q = query($s);
9340: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
9341: 
9342: 				$d = fetch_assoc($q);
9343: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
9344: 				if ($d['client'] != $_SESSION[UID]) 
9345: 				{
9346: 					return $this->showError('404', 'error', 'user is not author');
9347: 				}
9348: 	
9349: 				$s = "SELECT 
9350: 						`value`
9351: 					FROM `users_prop_items_varchar`
9352: 					WHERE 
9353: 						`id_user` = '" . $_SESSION[UID] . "' AND `id_users_prop` = '3'
9354: 					LIMIT 1
9355: 					";
9356: 
9357: 				$q = query($s);
9358: 				if ($q === false) return $this->showError('404', 'error', 'user select failed');
9359: 				
9360: 				$d_u = fetch_assoc($q);
9361: 				if (isset($d_u['value'])) $user_service_profile = $d_u['value'];
9362: 
9363: 				$d['options'] = json_decode($d['options'],true);
9364: 
9365: 				switch ($d['id_order_status']) {
9366: 					case '1':
9367: 						$allowed_fields = array(
9368: 												'b_start_address'			=>		'from',				
9369: 												'b_start_latitude'			=>		'from_lat',
9370: 												'b_start_longitude'			=>		'from_lng',
9371: 												'b_destination_address'		=>		'to',
```

### `archive_17012026_1259/taxi/models/api.php:10644`
```php
10634: 		}
10635: 
10636: 		private function prepareForDriverCode($str = "")
10637: 		{
10638: 			return strtr(strtolower((string)$str),array('d'=>'a','o'=>'0','q'=>'0','i'=>'1','l'=>'1'));
10639: 		}
10640: 
10641: 		public function selectFavoriteUser($id_user = "")
10642: 		{
10643: 			if (empty($_SESSION[UID])) {
10644: 				return $this->showError('404', 'error', 'unauthorized access');
10645: 			}
10646: 			$sql_add = "
10647: 					`users`.`phone` as u_phone,
10648: 					`users`.`phone_is_verified` as u_phone_checked,
10649: 					`users`.`email` as u_email,
10650: 					`users`.`birthday_date` as u_birthday,
10651: 					`users`.`referral_code` as ref_code,
10652: 					`users`.`referrer` as referrer_u_id,
10653: 					`tg` as u_tg,
10654: 					`tg_is_verified` as u_tg_checked,
10655: 					`id_user_upper` as u_upper,
10656: 					`wa` as u_wa,
10657: 					`wa_is_verified` as u_wa_checked,";
10658: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10659: 			
10660: 			{
10661: 				$id_user = $_SESSION[UID];
10662: 				if ($this->id_role != 4) $sql_add = "";
10663: 			}
10664: 			else
10665: 			{	
10666: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10667: 			}
10668: 
10669: 			$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:10658`
```php
10648: 					`users`.`phone_is_verified` as u_phone_checked,
10649: 					`users`.`email` as u_email,
10650: 					`users`.`birthday_date` as u_birthday,
10651: 					`users`.`referral_code` as ref_code,
10652: 					`users`.`referrer` as referrer_u_id,
10653: 					`tg` as u_tg,
10654: 					`tg_is_verified` as u_tg_checked,
10655: 					`id_user_upper` as u_upper,
10656: 					`wa` as u_wa,
10657: 					`wa_is_verified` as u_wa_checked,";
10658: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10659: 			
10660: 			{
10661: 				$id_user = $_SESSION[UID];
10662: 				if ($this->id_role != 4) $sql_add = "";
10663: 			}
10664: 			else
10665: 			{	
10666: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10667: 			}
10668: 
10669: 			$s = "SELECT 
10670: 					`users`.`id_user` as u_id,
10671: 					`users`.`id_role` as u_role,
10672: 					`users`.`name` as u_name,
10673: 					`users`.`family` as u_family,
10674: 					`users`.`middle` as u_middle,
10675: 					" . $sql_add . "
10676: 					`users`.`photo_link` as u_photo,
10677: 					`users`.`id_lang` as u_lang,
10678: 					`users`.`currency` as u_currency,
10679: 					`users`.`id_city` as u_city,
10680: 					`users`.`tips` as u_tips,
10681: 					`users`.`language_skills` as u_lang_skills,
10682: 					`users`.`description` as u_description,
10683: 					`users`.`id_navigation` as u_gps_software,
```

### `archive_17012026_1259/taxi/models/api.php:10771`
```php
10761: 
10762: 				$out['user'][$d['u_id']] = $d;
10763: 			}
10764: 
10765: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
10766: 
10767: 			return array(
10768: 				'code' 		=>	'200',
10769: 				'status' 	=>	'success',		
10770: 				'data' 		=>	$out,
10771: 				'auth_user' =>	array(
10772: 									'u_id' => $_SESSION[UID],
10773: 									'u_name' => $_SESSION['name'],
10774: 									'u_family' => $_SESSION['family'],
10775: 									'u_middle' => $_SESSION['middle'],
10776: 									'u_email' => $_SESSION['email'],
10777: 									'u_phone' => $_SESSION['phone'],
10778: 									'u_role' => $_SESSION['id_role'],
10779: 									'u_a_role' => $this->id_role,
10780: 									'u_check_state' => $_SESSION['id_verification_status'],
10781: 									'u_ban' => $_SESSION['user_ban_status'],
10782: 									'u_active' => $_SESSION['active'],
10783: 									'u_photo' => $_SESSION['photo_link'],
10784: 									'u_birthday' => $_SESSION['birthday_date'],
10785: 									'u_lang' => $_SESSION['id_lang'],
10786: 									'u_currency' => $_SESSION['currency'],
10787: 									'u_gps_software' => $_SESSION['id_navigation']
10788: 								)
10789: 			);
10790: 		}
10791: 		
10792: 		public function addFavoriteUser($id_user = "", $id_favorite = "")
10793: 		{
10794: 			if (empty($_SESSION[UID])) {
10795: 				return $this->showError('404', 'error', 'unauthorized access');
10796: 			}
```

### `archive_17012026_1259/taxi/models/api.php:10795`
```php
10785: 									'u_lang' => $_SESSION['id_lang'],
10786: 									'u_currency' => $_SESSION['currency'],
10787: 									'u_gps_software' => $_SESSION['id_navigation']
10788: 								)
10789: 			);
10790: 		}
10791: 		
10792: 		public function addFavoriteUser($id_user = "", $id_favorite = "")
10793: 		{
10794: 			if (empty($_SESSION[UID])) {
10795: 				return $this->showError('404', 'error', 'unauthorized access');
10796: 			}
10797: 
10798: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10799: 			{	
10800: 				$id_user = $_SESSION[UID];
10801: 			}
10802: 			else
10803: 			{	
10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10805: 				$s = "SELECT
10806: 						`id_user`
10807: 					FROM `users`
10808: 					WHERE `id_user` = '" . $id_user . "'
10809: 					LIMIT 1
10810: 					";
10811: 
10812: 				$q = query($s);
10813: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10814: 				$d = fetch_assoc($q);
10815: 				
10816: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10817: 			}
10818: 
10819: 			if (empty($id_favorite)) return $this->showError('404', 'error', 'empty favorite');
10820: 			$favorite = explode(',', $id_favorite);
```

### `archive_17012026_1259/taxi/models/api.php:10798`
```php
10788: 								)
10789: 			);
10790: 		}
10791: 		
10792: 		public function addFavoriteUser($id_user = "", $id_favorite = "")
10793: 		{
10794: 			if (empty($_SESSION[UID])) {
10795: 				return $this->showError('404', 'error', 'unauthorized access');
10796: 			}
10797: 
10798: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10799: 			{	
10800: 				$id_user = $_SESSION[UID];
10801: 			}
10802: 			else
10803: 			{	
10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10805: 				$s = "SELECT
10806: 						`id_user`
10807: 					FROM `users`
10808: 					WHERE `id_user` = '" . $id_user . "'
10809: 					LIMIT 1
10810: 					";
10811: 
10812: 				$q = query($s);
10813: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10814: 				$d = fetch_assoc($q);
10815: 				
10816: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10817: 			}
10818: 
10819: 			if (empty($id_favorite)) return $this->showError('404', 'error', 'empty favorite');
10820: 			$favorite = explode(',', $id_favorite);
10821: 			$favorite_count = count($favorite );
10822: 			if (in_array($id_user, $favorite))
10823: 			{
```

### `archive_17012026_1259/taxi/models/api.php:10882`
```php
10872: 
10873: 			return array(
10874: 				'code' 		=>	'200',
10875: 				'status' 	=>	'success'
10876: 			);
10877: 		}
10878: 		
10879: 		public function removeFavoriteUser($id_user = "", $id_favorite = "")
10880: 		{
10881: 			if (empty($_SESSION[UID])) {
10882: 				return $this->showError('404', 'error', 'unauthorized access');
10883: 			}
10884: 
10885: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10886: 			{	
10887: 				$id_user = $_SESSION[UID];
10888: 			}
10889: 			else
10890: 			{	
10891: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10892: 				$s = "SELECT
10893: 						`id_user`
10894: 					FROM `users`
10895: 					WHERE `id_user` = '" . $id_user . "'
10896: 					LIMIT 1
10897: 					";
10898: 
10899: 				$q = query($s);
10900: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10901: 				$d = fetch_assoc($q);
10902: 				
10903: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10904: 			}
10905: 
10906: 			$s = "DELETE
10907: 				FROM `users_favorite`
```

### `archive_17012026_1259/taxi/models/api.php:10885`
```php
10875: 				'status' 	=>	'success'
10876: 			);
10877: 		}
10878: 		
10879: 		public function removeFavoriteUser($id_user = "", $id_favorite = "")
10880: 		{
10881: 			if (empty($_SESSION[UID])) {
10882: 				return $this->showError('404', 'error', 'unauthorized access');
10883: 			}
10884: 
10885: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10886: 			{	
10887: 				$id_user = $_SESSION[UID];
10888: 			}
10889: 			else
10890: 			{	
10891: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10892: 				$s = "SELECT
10893: 						`id_user`
10894: 					FROM `users`
10895: 					WHERE `id_user` = '" . $id_user . "'
10896: 					LIMIT 1
10897: 					";
10898: 
10899: 				$q = query($s);
10900: 				if ($q === false) return $this->showError('404', 'error', 'database user select failed');
10901: 				$d = fetch_assoc($q);
10902: 				
10903: 				if (empty($d['id_user'])) return $this->showError('404', 'error', 'user ' . $id_user . ' not found');
10904: 			}
10905: 
10906: 			$s = "DELETE
10907: 				FROM `users_favorite`
10908: 				WHERE 
10909: 					`id_user` = '" . $id_user . "' AND `id_favorite` in (" . $id_favorite  . ")
10910: 				";
```

### `archive_17012026_1259/taxi/models/api.php:10924`
```php
10914: 
10915: 			return array(
10916: 				'code' 		=>	'200',
10917: 				'status' 	=>	'success'
10918: 			);
10919: 		}
10920: 
10921: 		public function selectReferralUrl($id_user = "")
10922: 		{
10923: 			if (empty($_SESSION[UID])) {
10924: 				return $this->showError('404', 'error', 'unauthorized access');
10925: 			}
10926: 
10927: 			$sql_limit = "LIMIT 1";
10928: 
10929: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10930: 			{
10931: 				$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
10932: 			}
10933: 			else
10934: 			{	
10935: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10936: 
10937: 				$sql_where = "AND `id_user` in (" . $id_user . ")";
10938: 
10939: 				$sql_limit = "LIMIT " . $this->limit_offset . ", " . $this->limit_row_count;
10940: 			}
10941: 
10942: 			$s = "SELECT 
10943: 					`id_user` as u_id
10944: 				FROM `users`
10945: 				WHERE
10946: 					`deleted` = '0'
10947: 				" . $sql_where . "
10948: 				" . $sql_limit . "
10949: 				";
```

### `archive_17012026_1259/taxi/models/api.php:10929`
```php
10919: 		}
10920: 
10921: 		public function selectReferralUrl($id_user = "")
10922: 		{
10923: 			if (empty($_SESSION[UID])) {
10924: 				return $this->showError('404', 'error', 'unauthorized access');
10925: 			}
10926: 
10927: 			$sql_limit = "LIMIT 1";
10928: 
10929: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
10930: 			{
10931: 				$sql_where = "AND `id_user` = '" . $_SESSION[UID] . "'";
10932: 			}
10933: 			else
10934: 			{	
10935: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
10936: 
10937: 				$sql_where = "AND `id_user` in (" . $id_user . ")";
10938: 
10939: 				$sql_limit = "LIMIT " . $this->limit_offset . ", " . $this->limit_row_count;
10940: 			}
10941: 
10942: 			$s = "SELECT 
10943: 					`id_user` as u_id
10944: 				FROM `users`
10945: 				WHERE
10946: 					`deleted` = '0'
10947: 				" . $sql_where . "
10948: 				" . $sql_limit . "
10949: 				";
10950: 
10951: 			$q = query($s);
10952: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
10953: 			
10954: 			$out = array('user' => array());
```

### `archive_17012026_1259/taxi/models/api.php:10976`
```php
10966: 
10967: 
10968: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
10969: 
10970: 
10971: 
10972: 			return array(
10973: 				'code' 		=>	'200',
10974: 				'status' 	=>	'success',		
10975: 				'data' 		=>	$out,
10976: 				'auth_user' =>	array(
10977: 									'u_id' => $_SESSION[UID],
10978: 									'u_name' => $_SESSION['name'],
10979: 									'u_family' => $_SESSION['family'],
10980: 									'u_middle' => $_SESSION['middle'],
10981: 									'u_email' => $_SESSION['email'],
10982: 									'u_phone' => $_SESSION['phone'],
10983: 									'u_role' => $_SESSION['id_role'],
10984: 									'u_a_role' => $this->id_role,
10985: 									'u_check_state' => $_SESSION['id_verification_status'],
10986: 									'u_ban' => $_SESSION['user_ban_status'],
10987: 									'u_active' => $_SESSION['active'],
10988: 									'u_photo' => $_SESSION['photo_link'],
10989: 									'u_birthday' => $_SESSION['birthday_date'],
10990: 									'u_lang' => $_SESSION['id_lang'],
10991: 									'u_currency' => $_SESSION['currency'],
10992: 									'u_gps_software' => $_SESSION['id_navigation']
10993: 								)
10994: 			);
10995: 		}
10996: 
10997: 		public function checkReferralCode($str = "")
10998: 		{
10999: 			if (empty($str)) return $this->showError('404', 'error', 'empty string');
11000: 
11001: 			$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:11069`
```php
11059: 				'code' 		=>	'200',
11060: 				'status' 	=>	'success',		
11061: 				'data' 		=>	$json
11062: 			);
11063: 		
11064: 		}
11065: 
11066: 		public function selectReferralUser($id_user = "")
11067: 		{
11068: 			if (empty($_SESSION[UID])) {
11069: 				return $this->showError('404', 'error', 'unauthorized access');
11070: 			}
11071: 			$sql_add = "
11072: 					`phone` as u_phone,
11073: 					`phone_is_verified` as u_phone_checked,
11074: 					`email` as u_email,
11075: 					`birthday_date` as u_birthday,
11076: 					`referral_code` as ref_code,
11077: 					`referrer` as referrer_u_id,
11078: 					`tg` as u_tg,
11079: 					`tg_is_verified` as u_tg_checked,
11080: 					`id_user_upper` as u_upper,
11081: 					`wa` as u_wa,
11082: 					`wa_is_verified` as u_wa_checked,";
11083: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
11084: 			{
11085: 				$id_user = $_SESSION[UID];
11086: 				if ($this->id_role != 4) $sql_add = "";
11087: 			}
11088: 			else
11089: 			{	
11090: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11091: 			}
11092: 
11093: 			$s = "SELECT 
11094: 					`id_user` as u_id,
```

### `archive_17012026_1259/taxi/models/api.php:11083`
```php
11073: 					`phone_is_verified` as u_phone_checked,
11074: 					`email` as u_email,
11075: 					`birthday_date` as u_birthday,
11076: 					`referral_code` as ref_code,
11077: 					`referrer` as referrer_u_id,
11078: 					`tg` as u_tg,
11079: 					`tg_is_verified` as u_tg_checked,
11080: 					`id_user_upper` as u_upper,
11081: 					`wa` as u_wa,
11082: 					`wa_is_verified` as u_wa_checked,";
11083: 			if (empty($id_user) || $id_user == 'authorized' || $id_user == $_SESSION[UID])
11084: 			{
11085: 				$id_user = $_SESSION[UID];
11086: 				if ($this->id_role != 4) $sql_add = "";
11087: 			}
11088: 			else
11089: 			{	
11090: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11091: 			}
11092: 
11093: 			$s = "SELECT 
11094: 					`id_user` as u_id,
11095: 					`id_role` as u_role,
11096: 					`name` as u_name,
11097: 					`family` as u_family,
11098: 					`middle` as u_middle,
11099: 					" . $sql_add . "
11100: 					`photo_link` as u_photo,
11101: 					`id_lang` as u_lang,
11102: 					`currency` as u_currency,
11103: 					`id_city` as u_city,
11104: 					`tips` as u_tips,
11105: 					`language_skills` as u_lang_skills,
11106: 					`description` as u_description,
11107: 					`id_navigation` as u_gps_software,
11108: 					`id_verification_status` as u_check_state,
```

### `archive_17012026_1259/taxi/models/api.php:11180`
```php
11170: 				$d['u_active'] = (int)$d['u_active'];			
11171: 				$d['out_drive'] = (int)$d['out_drive'];	
11172: 				$d['u_details'] = json_decode($d['u_details'],true);
11173: 				$out['user'][$d['u_id']] = $d;
11174: 			}
11175: 			if (empty($this->associativeArray)) $out['user'] =  array_values($out['user']);
11176: 			return array(
11177: 				'code' 		=>	'200',
11178: 				'status' 	=>	'success',		
11179: 				'data' 		=>	$out,
11180: 				'auth_user' =>	array(
11181: 									'u_id' => $_SESSION[UID],
11182: 									'u_name' => $_SESSION['name'],
11183: 									'u_family' => $_SESSION['family'],
11184: 									'u_middle' => $_SESSION['middle'],
11185: 									'u_email' => $_SESSION['email'],
11186: 									'u_phone' => $_SESSION['phone'],
11187: 									'u_role' => $_SESSION['id_role'],
11188: 									'u_a_role' => $this->id_role,
11189: 									'u_check_state' => $_SESSION['id_verification_status'],
11190: 									'u_ban' => $_SESSION['user_ban_status'],
11191: 									'u_active' => $_SESSION['active'],
11192: 									'u_photo' => $_SESSION['photo_link'],
11193: 									'u_birthday' => $_SESSION['birthday_date'],
11194: 									'u_lang' => $_SESSION['id_lang'],
11195: 									'u_currency' => $_SESSION['currency'],
11196: 									'u_gps_software' => $_SESSION['id_navigation']
11197: 								)
11198: 			);
11199: 		}
11200: 
11201: 		public function setOrderTips($id_order = "", $b_tips = NULL, $c_tips = NULL)
11202: 		{
11203: 			if (empty($_SESSION[UID])) {
11204: 				return $this->showError('404', 'error', 'unauthorized access');
11205: 			}
```

### `archive_17012026_1259/taxi/models/api.php:11204`
```php
11194: 									'u_lang' => $_SESSION['id_lang'],
11195: 									'u_currency' => $_SESSION['currency'],
11196: 									'u_gps_software' => $_SESSION['id_navigation']
11197: 								)
11198: 			);
11199: 		}
11200: 
11201: 		public function setOrderTips($id_order = "", $b_tips = NULL, $c_tips = NULL)
11202: 		{
11203: 			if (empty($_SESSION[UID])) {
11204: 				return $this->showError('404', 'error', 'unauthorized access');
11205: 			}
11206: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
11207: 			{
11208: 				return $this->showError('404', 'error', 'wrong user role');
11209: 			}
11210: 
11211: 			if ($this->id_role == 1 || $this->id_role == 5)
11212: 			{
11213: 				if ($b_tips === NULL) return $this->showError('404', 'error', 'null b_tips');
11214: 				$tips = $b_tips;
11215: 
11216: 				$s = "SELECT
11217: 						`id_order`,
11218: 						`client`,
11219: 						`id_order_status`,
11220: 						`tips`
11221: 					FROM `order` 		
11222: 					WHERE	
11223: 						`id_order` = '" . $id_order . "'
11224: 					LIMIT 1
11225: 					";
11226: 
11227: 				$q = query($s);
11228: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
11229: 
```

### `archive_17012026_1259/taxi/models/api.php:11235`
```php
11225: 					";
11226: 
11227: 				$q = query($s);
11228: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
11229: 
11230: 				$d = fetch_assoc($q);
11231: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
11232: 				if ($d['id_order_status'] != 4) return $this->showError('404', 'error', 'wrong booking state');
11233: 				if ($d['client'] != $_SESSION[UID]) 
11234: 				{
11235: 					return $this->showError('404', 'error', 'user is not author');
11236: 				}
11237: 				if ($d['tips'] !== NULL) return $this->showError('404', 'error', 'c_tips already inputed');
11238: 
11239: 				$s = "UPDATE `order`
11240: 					SET 
11241: 						`tips` = '" . $tips  . "',
11242: 						`last_edit_datetime` = now(),
11243: 						`last_edit_user` = '" .  $_SESSION[UID] . "'
11244: 					WHERE
11245: 						`id_order` = '" . $id_order . "' AND `tips` IS NULL
11246: 					";
11247: 
11248: 				query($s);
11249: 		
11250: 				global $link;
11251: 				if (mysqli_affected_rows($link) === -1) 
11252: 				{
11253: 					return $this->showError('404', 'error', 'database update failed');
11254: 				}
11255: 				elseif (mysqli_affected_rows($link) === 0) 
11256: 				{
11257: 					return $this->showError('404', 'error', 'modified data not found');
11258: 				}
11259: 			}
11260: 			else
```

### `archive_17012026_1259/taxi/models/api.php:11344`
```php
11334: 				'code' 		=>	'200',
11335: 				'status' 	=>	'success'
11336: 			);
11337: 		}
11338: 
11339: 		public $id_role = NULL;
11340: 
11341: 		public function selectUsersCargoCase($id_user = "")
11342: 		{
11343: /*			if (empty($_SESSION[UID])) {
11344: 				return $this->showError('404', 'error', 'unauthorized access');
11345: 			}*/
11346: 
11347: 			$s = "SELECT
11348: 					`id_cargo_case`
11349: 				FROM `cargo_case_users` 		
11350: 				WHERE	
11351: 					`id_user` = '" . $id_user . "'
11352: 				";
11353: 
11354: 			$q = query($s);
11355: 			/*if ($q === false) return $this->showError('404', 'error', 'database select failed');*/
11356: 
11357: 			$out = array();
11358: 			while ($d = fetch_assoc($q))
11359: 			{
11360: 				$out[$d['id_cargo_case']] = true;
11361: 			}
11362: 			return $out;
11363: 		}
11364: 
11365: 		public function createTrip($id_user = "", $data = "", $langs = array(), $s_data = array())
11366: 		{
11367: 			if (empty($_SESSION[UID])) {
11368: 				return $this->showError('404', 'error', 'unauthorized access');
11369: 			}
```

### `archive_17012026_1259/taxi/models/api.php:11368`
```php
11358: 			while ($d = fetch_assoc($q))
11359: 			{
11360: 				$out[$d['id_cargo_case']] = true;
11361: 			}
11362: 			return $out;
11363: 		}
11364: 
11365: 		public function createTrip($id_user = "", $data = "", $langs = array(), $s_data = array())
11366: 		{
11367: 			if (empty($_SESSION[UID])) {
11368: 				return $this->showError('404', 'error', 'unauthorized access');
11369: 			}
11370: 
11371: 			if (empty($id_user) || $id_user == $_SESSION[UID])
11372: 			{
11373: 				if ($this->id_role != 2)
11374: 				{
11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
11376: 				}
11377: 				else
11378: 				{
11379: 					if ($_SESSION['id_verification_status'] != 2)
11380: 					{
11381: 						return $this->showError('404', 'error', 'wrong user check state');
11382: 					}
11383: 					if (!empty($_SESSION['user_ban_status']['order']))
11384: 					{
11385: 						return $this->showError('404', 'error', 'user banned');
11386: 					}
11387: 				}
11388: 				$id_user = $_SESSION[UID];
11389: 			}
11390: 			else
11391: 			{
11392: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
11393: 				$s = "SELECT 
```

### `archive_17012026_1259/taxi/models/api.php:12133`
```php
12123: 			return array(
12124: 				'code' 		=>	'200',
12125: 				'status' 	=>	'success',
12126: 				'data' 		=>	$out
12127: 			);
12128: 		}
12129: 
12130: 		public function selectTrip($id_trip = "", $type = NULL, $id_order = "", $filter = NULL, $fields = 0, $raw_price = false, $with_import = false, $price_time_functions = array(), $aggregators = array())
12131: 		{
12132: 			if (empty($_SESSION[UID])) {
12133: 				if ($type !== NULL) return $this->showError('404', 'error', 'unauthorized access');
12134: 			}
12135: 			else
12136: 			{
12137: 				if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 4 && $this->id_role != 5)
12138: 				{
12139: 					if ($type !== NULL) return $this->showError('404', 'error', 'wrong user role');
12140: 				}
12141: 			}
12142: 
12143: 			$sql_left_join = $orders = $cart = '';
12144: 			if (empty($id_trip))
12145: 			{
12146: 				$sql_where = '1=1';
12147: 			}
12148: 			else
12149: 			{
12150: 				$sql_where = "`trip`.`id_trip` in (" . $id_trip . ")";
12151: 			}
12152: 			if (empty($with_import) && empty($this->constant['select_trip_with_import'])) $sql_where .= ' AND `trip`.`id_aggregator` IS NULL';
12153: 			$cart_sold_seats = "";
12154: 			$field_flag = array();
12155: 			if (!empty($fields))
12156: 			{
12157: 				$field_arr	= get_field_arr(empty($type) ? 'trip' : "{$type}Trip",$this->id_role);
12158: 				$bin_arr = get_bin_arr();
```

### `archive_17012026_1259/taxi/models/api.php:12849`
```php
12839: 				add_time_zone($d['t_complete_real_datetime']);
12840: 				add_time_zone($d['t_edit_datetime']);
12841: 				$d['t_looking_for_clients'] = (int)$d['t_looking_for_clients'];	
12842: 				$d['t_canceled'] = (int)$d['t_canceled'];	
12843: 				$out['trip'][$d['t_id']] = $d;
12844: 			}
12845: 			if (empty($this->associativeArray)) $out['trip'] =  array_values($out['trip']);
12846: 
12847: 			if (!empty($_SESSION[UID]))
12848: 			{
12849: 				$auth_user = array(
12850: 									'u_id' => $_SESSION[UID],
12851: 									'u_name' => $_SESSION['name'],
12852: 									'u_family' => $_SESSION['family'],
12853: 									'u_middle' => $_SESSION['middle'],
12854: 									'u_email' => $_SESSION['email'],
12855: 									'u_phone' => $_SESSION['phone'],
12856: 									'u_role' => $_SESSION['id_role'],
12857: 									'u_a_role' => $this->id_role,
12858: 									'u_check_state' => $_SESSION['id_verification_status'],
12859: 									'u_ban' => $_SESSION['user_ban_status'],
12860: 									'u_active' => $_SESSION['active'],
12861: 									'u_photo' => $_SESSION['photo_link'],
12862: 									'u_birthday' => $_SESSION['birthday_date'],
12863: 									'u_lang' => $_SESSION['id_lang'],
12864: 									'u_currency' => $_SESSION['currency'],
12865: 									'u_gps_software' => $_SESSION['id_navigation']
12866: 				);
12867: 			}
12868: 			else
12869: 			{
12870: 				$auth_user = array();
12871: 			}
12872: 			return array(
12873: 				'code' 		=>	'200',
12874: 				'status' 	=>	'success',		
```

### `archive_17012026_1259/taxi/models/api.php:12870`
```php
12860: 									'u_active' => $_SESSION['active'],
12861: 									'u_photo' => $_SESSION['photo_link'],
12862: 									'u_birthday' => $_SESSION['birthday_date'],
12863: 									'u_lang' => $_SESSION['id_lang'],
12864: 									'u_currency' => $_SESSION['currency'],
12865: 									'u_gps_software' => $_SESSION['id_navigation']
12866: 				);
12867: 			}
12868: 			else
12869: 			{
12870: 				$auth_user = array();
12871: 			}
12872: 			return array(
12873: 				'code' 		=>	'200',
12874: 				'status' 	=>	'success',		
12875: 				'data' 		=>	$out,
12876: 				'auth_user' =>	$auth_user
12877: 			);
12878: 		}
12879: 
12880: 		public function editTrip($data = "", $id_trip = "")
12881: 		{
12882: 			if (empty($_SESSION[UID])) {
12883: 				return $this->showError('404', 'error', 'unauthorized access');
12884: 			}
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
12889: 			}
12890: 
12891: 			if (empty($data)) 
12892: 			{
12893: 				return $this->showError('404', 'error', 'empty data');
12894: 			}
12895: 
```

### `archive_17012026_1259/taxi/models/api.php:12876`
```php
12866: 				);
12867: 			}
12868: 			else
12869: 			{
12870: 				$auth_user = array();
12871: 			}
12872: 			return array(
12873: 				'code' 		=>	'200',
12874: 				'status' 	=>	'success',		
12875: 				'data' 		=>	$out,
12876: 				'auth_user' =>	$auth_user
12877: 			);
12878: 		}
12879: 
12880: 		public function editTrip($data = "", $id_trip = "")
12881: 		{
12882: 			if (empty($_SESSION[UID])) {
12883: 				return $this->showError('404', 'error', 'unauthorized access');
12884: 			}
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
12889: 			}
12890: 
12891: 			if (empty($data)) 
12892: 			{
12893: 				return $this->showError('404', 'error', 'empty data');
12894: 			}
12895: 
12896: 			$data = json_decode($data,true);
12897: 			
12898: 			if (empty($data) || !is_array($data)) 
12899: 			{
12900: 				return $this->showError('404', 'error', 'wrong data');
12901: 			}
```

### `archive_17012026_1259/taxi/models/api.php:12883`
```php
12873: 				'code' 		=>	'200',
12874: 				'status' 	=>	'success',		
12875: 				'data' 		=>	$out,
12876: 				'auth_user' =>	$auth_user
12877: 			);
12878: 		}
12879: 
12880: 		public function editTrip($data = "", $id_trip = "")
12881: 		{
12882: 			if (empty($_SESSION[UID])) {
12883: 				return $this->showError('404', 'error', 'unauthorized access');
12884: 			}
12885: 
12886: 			if ($this->id_role != 2 && $this->id_role != 4)
12887: 			{
12888: 				return $this->showError('404', 'error', 'wrong user role');
12889: 			}
12890: 
12891: 			if (empty($data)) 
12892: 			{
12893: 				return $this->showError('404', 'error', 'empty data');
12894: 			}
12895: 
12896: 			$data = json_decode($data,true);
12897: 			
12898: 			if (empty($data) || !is_array($data)) 
12899: 			{
12900: 				return $this->showError('404', 'error', 'wrong data');
12901: 			}
12902: 
12903: 			$allowed_fields = array(
12904: 									't_start_address'			=>		'from',
12905: 									't_start_latitude'			=>		'from_lat',
12906: 									't_start_longitude'			=>		'from_lng',
12907: 									't_destination_address'		=>		'to',
12908: 									't_destination_latitude'	=>		'to_lat',
```

### `archive_17012026_1259/taxi/models/api.php:12984`
```php
12974: 				FOR UPDATE
12975: 				";
12976: 
12977: 			$q = query($s);
12978: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
12979: 
12980: 			$d = fetch_assoc($q);
12981: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
12982: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
12983: 			{
12984: 				return $this->showError('404', 'error', 'user is not author');
12985: 			}
12986: 			if ($d['complete_datetime'] != "0000-00-00 00:00:00") return $this->showError('404', 'error', 'completed trip');
12987: 			if (!empty($d['canceled'])) return $this->showError('404', 'error', 'canceled trip');
12988: 
12989: 			$d['json'] = json_decode($d['json'],true);
12990: 
12991: 			if (!empty($d['orders']))
12992: 			{
12993: 				$d['orders'] = explode(chr(1),$d['orders']);
12994: 				foreach ($d['orders'] as $key=>$value)
12995: 				{
12996: 					$d['orders'][$key] = array();
12997: 					list(
12998: 						$d['orders'][$key]['id_order'],
12999: 						$d['orders'][$key]['offer_order_datetime'],
13000: 						$d['orders'][$key]['select_trip_datetime'],
13001: 						$d['orders'][$key]['id_order_status'],
13002: 						$d['orders'][$key]['id_order_driver_status']
13003: 						)= explode(chr(0),$value);
13004: 
13005: 					if ($d['orders'][$key]['id_order_driver_status'] === chr(2))
13006: 					{
13007: 						$d['orders'][$key]['id_order_driver_status'] = NULL;
13008: 					}
13009: 				}
```

### `archive_17012026_1259/taxi/models/api.php:13570`
```php
13560: 				'data' 		=>	array(
13561: 									'affected_fields' 	=> 	$affected_fields,
13562: 									'forbidden_fields'	=>	$forbidden_fields
13563: 								)
13564: 			);
13565: 		}
13566: 
13567: 		public function offerOrder($id_order = "", $id_user = "", $trips = "")
13568: 		{
13569: 			if (empty($_SESSION[UID])) {
13570: 				return $this->showError('404', 'error', 'unauthorized access');
13571: 			}
13572: 			if ($this->id_role != 1 && $this->id_role != 5)
13573: 			{
13574: 				return $this->showError('404', 'error', 'wrong user role');
13575: 			}
13576: 			if ($id_user == $_SESSION[UID]) return $this->showError('404', 'error', 'trying to offer yourself');
13577: 			if (empty($id_order)) return $this->showError('404', 'error', 'empty booking');
13578: 			if (empty($id_user)) return $this->showError('404', 'error', 'empty driver');
13579: 
13580: 			$s = "SELECT
13581: 					`id_order`,
13582: 					`client`,
13583: 					`id_order_status`,
13584: 					GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
13585: 						. "',`order_driver`.`id_user`,NULL)) as u_id
13586: 				FROM `order` 	
13587: 				LEFT JOIN `order_driver` USING (`id_order`)	
13588: 				WHERE	
13589: 					`id_order` = '" . $id_order . "'
13590: 				LIMIT 1
13591: 				";
13592: 
13593: 			$q = query($s);
13594: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
13595: 
```

### `archive_17012026_1259/taxi/models/api.php:13601`
```php
13591: 				";
13592: 
13593: 			$q = query($s);
13594: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
13595: 
13596: 			$d = fetch_assoc($q);
13597: 			if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
13598: 			if ($d['id_order_status'] == 3 || $d['id_order_status'] == 4) return $this->showError('404', 'error', 'wrong booking state');
13599: 			if ($d['client'] != $_SESSION[UID]) 
13600: 			{
13601: 				return $this->showError('404', 'error', 'user is not author');
13602: 			}
13603: 			if (!empty($d['u_id'])) 
13604: 			{
13605: 				return $this->showError('404', 'error', $id_user . ' is performer');
13606: 			}
13607: 
13608: 			$s = "SELECT
13609: 					`id_user`
13610: 				FROM `users` 		
13611: 				WHERE	
13612: 					`id_user` = '" . $id_user. "'
13613: 				LIMIT 1
13614: 				";
13615: 
13616: 			$q = query($s);
13617: 			if ($q === false) return $this->showError('404', 'error', 'select of database failed');
13618: 
13619: 			$d = fetch_assoc($q);
13620: 			if (empty($d['id_user'])) return $this->showError('404', 'error', 'driver not found');
13621: 			
13622: 			$q = query("BEGIN");
13623: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
13624: 
13625: 			$s = "INSERT IGNORE INTO `order_driver_select`
13626: 					SET
```

### `archive_17012026_1259/taxi/models/api.php:13658`
```php
13648: 					FROM `trip` 		
13649: 					WHERE	
13650: 						`id_trip` in (" . $trips . ") AND `driver` = '" . $id_user . "'
13651: 					";
13652: 
13653: 				$q = query($s);
13654: 				if ($q === false) return $this->showError('404', 'error', 'select failed');
13655: 
13656: 				$d = fetch_assoc($q);
13657: 				$trips = explode(',', $trips);
13658: 				if ($d['trips_count'] != count($trips)) return $this->showError('404', 'error', 'driver is not trip author');
13659: 				
13660: 				$s = array();
13661: 				foreach ($trips as $id_trip)
13662: 				{
13663: 					$s[] = "('" . $id_order . "', '" . $id_trip . "', now(), now())";
13664: 				}
13665: 				$s = "INSERT INTO `order_trip` (`id_order`,  `id_trip`, `create_datetime`, `offer_order_datetime`) VALUES " . implode(",", $s) . "ON DUPLICATE KEY UPDATE `offer_order_datetime` = IF(`offer_order_datetime` = 0,now(),`offer_order_datetime`)";
13666: 
13667: 				$q = query($s);
13668: 				if ($q === false) return $this->showError('404', 'error', 'insert in database failed');
13669: 			}
13670: 	
13671: 			$s = "UPDATE `order`
13672: 				SET
13673: 					`last_edit_datetime` = now(),
13674: 					`last_edit_user` = '" .  $_SESSION[UID] . "'
13675: 				WHERE
13676: 					`id_order` = '" . $id_order . "'
13677: 				";
13678: 
13679: 			$q = query($s);
13680: 			if ($q === false) return $this->showError('404', 'error', 'database timestamp update failed');
13681: 
13682: 			$q = query("COMMIT");
13683: 			if ($q === false) return $this->showError('404', 'error', 'commit query failed');
```

### `archive_17012026_1259/taxi/models/api.php:13693`
```php
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
13717: 
13718: 			@$file = json_decode($file,true);
```

### `archive_17012026_1259/taxi/models/api.php:13715`
```php
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
13725: 			if (isset($file['u_id']))
13726: 			{
13727: 				$id_user = trim($file['u_id']);	
13728: 				if ($id_user != $_SESSION[UID])
13729: 				{
13730: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
13731: 
13732: 					$s = "SELECT
13733: 							`id_user`
13734: 						FROM `users` 		
13735: 						WHERE	
13736: 							`id_user` = '" . $id_user. "'
13737: 						LIMIT 1
13738: 						";
13739: 
13740: 					$q = query($s);
```

### `archive_17012026_1259/taxi/models/api.php:14055`
```php
14045: 			$q = query($s);
14046: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
14047: 
14048: 			$d = fetch_assoc($q);
14049: 			if (empty($d['id_dropbox_link'])) return $this->showError('404', 'error', 'dropbox link not found');
14050: 			if (!empty($d['deleted']))return $this->showError('404', 'error', 'deleted dropbox link');
14051: 			if (empty($_SESSION[UID]))
14052: 			{
14053: 				if ($d['private'] != -1)
14054: 				{
14055: 					return $this->showError('404', 'error', 'unauthorized access');
14056: 				}
14057: 			}
14058: 			else
14059: 			{
14060: 				if (isset($d['private']) && $d['private'] == 1 && empty($d['user']))
14061: 				{
14062: 					return $this->showError('404', 'error', 'not enough rights');
14063: 				}
14064: 			}
14065: 
14066: 			$json = json_decode($d['json'],true);
14067: 			$response = download_from_dropbox($json['name_upload'],$id_dropbox_link,$json['type']);
14068: 			if (!empty($response['error'])) return $this->showError('404', 'error', $response['error']);
14069: 		}
14070: 
14071: 		public function updateDropbox($code = '', $hash = '')
14072: 		{
14073: 			if ($hash === md5('checking' . md5(API_KEY)))
14074: 			{
14075: 				$res = update_dropbox($code);
14076: 				if (!empty($res['error']))
14077: 				{
14078: 					return $this->showError('404', 'error', $res['error']);
14079: 				}
14080: 				return array(
```

### `archive_17012026_1259/taxi/models/api.php:14343`
```php
14333: 			return array(
14334: 				'code' 		=>	'200',
14335: 				'status' 	=>	'success',		
14336: 				'data' 		=>	$json
14337: 			);
14338: 		}
14339: 
14340: 		public function editData($data = 0, $s_date = array(), $s_date_private = array(), $s_data_stt = array(), $s_data_sc = array())
14341: 		{
14342: 			if (empty($_SESSION[UID])) {
14343: 				return $this->showError('404', 'error', 'unauthorized access');
14344: 			}
14345: 
14346: 			if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
14347: 			{
14348: 				return $this->showError('404', 'error', 'wrong user check state');
14349: 			}
14350: 			
14351: 			if (empty($data)) 
14352: 			{
14353: 				return $this->showError('404', 'error', 'empty data');
14354: 			}
14355: 
14356: 			$data = json_decode($data,true);
14357: 			
14358: 			if (empty($data) || !is_array($data)) 
14359: 			{
14360: 				return $this->showError('404', 'error', 'wrong data');
14361: 			}
14362: 
14363: 			$data_template = array(
14364: 				'langs'	=>	array(
14365: 					'table'			=>	'lang',
14366: 					'id'			=>	'id_lang',
14367: 					'data_suffix'	=>	'',
14368: 					'allowed_fields'=>	array(
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
14424: 						'json'	=>		array(
14425: 												'name'	=>	'json',
14426: 												'NULL'	=>	false
14427: 											),
14428: 						'active'	=>		array(
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
14425: 												'name'	=>	'json',
14426: 												'NULL'	=>	false
14427: 											),
14428: 						'active'	=>		array(
14429: 												'name'	=>	'active',
```

### `archive_17012026_1259/taxi/models/api.php:14419`
```php
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
14433: 					)
14434: 				),
14435: 				'address_objects'	=>	array(
14436: 					'table'			=>	'address_object',
14437: 					'id'			=>	'id_address_object_type',
14438: 					'data_suffix'	=>	'',					
14439: 					'auto_fields'	=>	array(
14440: 						'timestamp_edited' => 'last_edit_int_timestamp',
14441: 						'timestamp_created' => 'create_int_timestamp',
14442: 						'e_u_id' => 'last_edit_user',
14443: 						'c_u_id' => 'create_user'
14444: 					),
```

### `archive_17012026_1259/taxi/models/api.php:14420`
```php
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
14433: 					)
14434: 				),
14435: 				'address_objects'	=>	array(
14436: 					'table'			=>	'address_object',
14437: 					'id'			=>	'id_address_object_type',
14438: 					'data_suffix'	=>	'',					
14439: 					'auto_fields'	=>	array(
14440: 						'timestamp_edited' => 'last_edit_int_timestamp',
14441: 						'timestamp_created' => 'create_int_timestamp',
14442: 						'e_u_id' => 'last_edit_user',
14443: 						'c_u_id' => 'create_user'
14444: 					),
14445: 					'allowed_fields'=>	array(
```

### `archive_17012026_1259/taxi/models/api.php:14768`
```php
14758: 												'name'	=>	'active',
14759: 												'NULL'	=>	false,
14760: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14761: 											),
14762: 						'json'	=>		array(
14763: 												'name'	=>	'json',
14764: 												'NULL'	=>	false
14765: 											)
14766: 					)
14767: 				),
14768: 				'contact_login_types'	=>	array(
14769: 					'table'			=>	'contact_login_type',
14770: 					'id'			=>	'id_contact_login_type',
14771: 					'data_suffix'	=>	'',					
14772: 					'auto_fields'	=>	array(
14773: 						'timestamp_edited' => 'last_edit_int_timestamp',
14774: 						'timestamp_created' => 'create_int_timestamp',
14775: 						'e_u_id' => 'last_edit_user',
14776: 						'c_u_id' => 'create_user'
14777: 					),
14778: 					'allowed_fields'=>	array(
14779: 						'json'	=>		array(
14780: 												'name'	=>	'json',
14781: 												'NULL'	=>	false
14782: 											),
14783: 						'active'	=>		array(
14784: 												'name'	=>	'active',
14785: 												'NULL'	=>	false,
14786: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14787: 											)
14788: 					)
14789: 				),
14790: 				'contact_classes'	=>	array(
14791: 					'table'			=>	'contact_type',
14792: 					'id'			=>	'id_contact_type',
14793: 					'data_suffix'	=>	'',
```

### `archive_17012026_1259/taxi/models/api.php:14769`
```php
14759: 												'NULL'	=>	false,
14760: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14761: 											),
14762: 						'json'	=>		array(
14763: 												'name'	=>	'json',
14764: 												'NULL'	=>	false
14765: 											)
14766: 					)
14767: 				),
14768: 				'contact_login_types'	=>	array(
14769: 					'table'			=>	'contact_login_type',
14770: 					'id'			=>	'id_contact_login_type',
14771: 					'data_suffix'	=>	'',					
14772: 					'auto_fields'	=>	array(
14773: 						'timestamp_edited' => 'last_edit_int_timestamp',
14774: 						'timestamp_created' => 'create_int_timestamp',
14775: 						'e_u_id' => 'last_edit_user',
14776: 						'c_u_id' => 'create_user'
14777: 					),
14778: 					'allowed_fields'=>	array(
14779: 						'json'	=>		array(
14780: 												'name'	=>	'json',
14781: 												'NULL'	=>	false
14782: 											),
14783: 						'active'	=>		array(
14784: 												'name'	=>	'active',
14785: 												'NULL'	=>	false,
14786: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14787: 											)
14788: 					)
14789: 				),
14790: 				'contact_classes'	=>	array(
14791: 					'table'			=>	'contact_type',
14792: 					'id'			=>	'id_contact_type',
14793: 					'data_suffix'	=>	'',
14794: 					'allowed_fields'	=>	array()
```

### `archive_17012026_1259/taxi/models/api.php:14770`
```php
14760: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14761: 											),
14762: 						'json'	=>		array(
14763: 												'name'	=>	'json',
14764: 												'NULL'	=>	false
14765: 											)
14766: 					)
14767: 				),
14768: 				'contact_login_types'	=>	array(
14769: 					'table'			=>	'contact_login_type',
14770: 					'id'			=>	'id_contact_login_type',
14771: 					'data_suffix'	=>	'',					
14772: 					'auto_fields'	=>	array(
14773: 						'timestamp_edited' => 'last_edit_int_timestamp',
14774: 						'timestamp_created' => 'create_int_timestamp',
14775: 						'e_u_id' => 'last_edit_user',
14776: 						'c_u_id' => 'create_user'
14777: 					),
14778: 					'allowed_fields'=>	array(
14779: 						'json'	=>		array(
14780: 												'name'	=>	'json',
14781: 												'NULL'	=>	false
14782: 											),
14783: 						'active'	=>		array(
14784: 												'name'	=>	'active',
14785: 												'NULL'	=>	false,
14786: 												'format'=>	function($val){return empty($val) ? 0 : 1;}
14787: 											)
14788: 					)
14789: 				),
14790: 				'contact_classes'	=>	array(
14791: 					'table'			=>	'contact_type',
14792: 					'id'			=>	'id_contact_type',
14793: 					'data_suffix'	=>	'',
14794: 					'allowed_fields'	=>	array()
14795: 				),
```

### `archive_17012026_1259/taxi/models/api.php:16733`
```php
16723: 			fields_for_langs($s_date['langs'],$data_template['cargo_categories'],'','name_',true);
16724: 			fields_for_langs($s_date['langs'],$data_template['cargo_categories'],'about_','description_',false);
16725: 			fields_for_langs($s_date['langs'],$data_template['cargo_value_types'],'','name_',true);
16726: 			fields_for_langs($s_date['langs'],$data_template['cargo_value_types'],'about_','description_',false);	
16727: 			fields_for_langs($s_date['langs'],$data_template['cart_block_statuses_sys'],'','name_',true);
16728: 			fields_for_langs($s_date['langs'],$data_template['cart_block_statuses_sys'],'about_','description_',false);
16729: 			fields_for_langs($s_date['langs'],$data_template['cities'],'','name_',true);
16730: 			fields_for_langs($s_date['langs'],$data_template['cities'],'about_','description_',false);
16731: 			fields_for_langs($s_date['langs'],$data_template['city_areas'],'','name_',true);
16732: 			fields_for_langs($s_date['langs'],$data_template['city_areas'],'about_','description_',false);	
16733: 			fields_for_langs($s_date['langs'],$data_template['contact_login_types'],'','name_',true);
16734: 			fields_for_langs($s_date['langs'],$data_template['contact_login_types'],'about_','description_',false);
16735: 			fields_for_langs($s_date['langs'],$data_template['contact_classes'],'','name_',true);
16736: 			fields_for_langs($s_date['langs'],$data_template['contact_classes'],'about_','description_',false);
16737: 			fields_for_langs($s_date['langs'],$data_template['countries'],'','country_name_',true);
16738: 			fields_for_langs($s_date['langs'],$data_template['currencies'],'','name_',true);
16739: 			fields_for_langs($s_date['langs'],$data_template['currencies'],'about_','country_',false);	
16740: 			fields_for_langs($s_date['langs'],$data_template['currency_account_statuses'],'','name_',true);
16741: 			fields_for_langs($s_date['langs'],$data_template['currency_account_statuses'],'about_','description_',false);
16742: 			fields_for_langs($s_date['langs'],$data_template['date_features'],'','name_',true);
16743: 			fields_for_langs($s_date['langs'],$data_template['date_features'],'about_','description_',false);
16744: 			fields_for_langs($s_date['langs'],$data_template['date_places'],'','name_',true);
16745: 			fields_for_langs($s_date['langs'],$data_template['date_places'],'about_','description_',false);
16746: 			fields_for_langs($s_date['langs'],$data_template['date_select'],'','name_',true);
16747: 			fields_for_langs($s_date['langs'],$data_template['date_select'],'about_','description_',false);
16748: 			fields_for_langs($s_date['langs'],$data_template['date_select_frequencies'],'','name_',true);
16749: 			fields_for_langs($s_date['langs'],$data_template['date_select_frequencies'],'about_','description_',false);
16750: 			fields_for_langs($s_date['langs'],$data_template['deal_statuses'],'','name_',true);
16751: 			fields_for_langs($s_date['langs'],$data_template['deal_statuses'],'about_','description_',false);
16752: 			fields_for_langs($s_date['langs'],$data_template['drivers_select_sorting'],'','name_',true);
16753: 			fields_for_langs($s_date['langs'],$data_template['drivers_select_sorting'],'about_','description_',false);
16754: 			fields_for_langs($s_date['langs'],$data_template['faqs'],'title_','name_',true);
16755: 			fields_for_langs($s_date['langs'],$data_template['faqs'],'content_','value_',false);
16756: 			fields_for_langs($s_date['langs'],$data_template['faq_groups'],'','name_',true);
16757: 			fields_for_langs($s_date['langs'],$data_template['favorite_addresses'],'','name_',true);
16758: 			fields_for_langs($s_date['langs'],$data_template['favorite_addresses'],'about_','description_',false);
```

### `archive_17012026_1259/taxi/models/api.php:16734`
```php
16724: 			fields_for_langs($s_date['langs'],$data_template['cargo_categories'],'about_','description_',false);
16725: 			fields_for_langs($s_date['langs'],$data_template['cargo_value_types'],'','name_',true);
16726: 			fields_for_langs($s_date['langs'],$data_template['cargo_value_types'],'about_','description_',false);	
16727: 			fields_for_langs($s_date['langs'],$data_template['cart_block_statuses_sys'],'','name_',true);
16728: 			fields_for_langs($s_date['langs'],$data_template['cart_block_statuses_sys'],'about_','description_',false);
16729: 			fields_for_langs($s_date['langs'],$data_template['cities'],'','name_',true);
16730: 			fields_for_langs($s_date['langs'],$data_template['cities'],'about_','description_',false);
16731: 			fields_for_langs($s_date['langs'],$data_template['city_areas'],'','name_',true);
16732: 			fields_for_langs($s_date['langs'],$data_template['city_areas'],'about_','description_',false);	
16733: 			fields_for_langs($s_date['langs'],$data_template['contact_login_types'],'','name_',true);
16734: 			fields_for_langs($s_date['langs'],$data_template['contact_login_types'],'about_','description_',false);
16735: 			fields_for_langs($s_date['langs'],$data_template['contact_classes'],'','name_',true);
16736: 			fields_for_langs($s_date['langs'],$data_template['contact_classes'],'about_','description_',false);
16737: 			fields_for_langs($s_date['langs'],$data_template['countries'],'','country_name_',true);
16738: 			fields_for_langs($s_date['langs'],$data_template['currencies'],'','name_',true);
16739: 			fields_for_langs($s_date['langs'],$data_template['currencies'],'about_','country_',false);	
16740: 			fields_for_langs($s_date['langs'],$data_template['currency_account_statuses'],'','name_',true);
16741: 			fields_for_langs($s_date['langs'],$data_template['currency_account_statuses'],'about_','description_',false);
16742: 			fields_for_langs($s_date['langs'],$data_template['date_features'],'','name_',true);
16743: 			fields_for_langs($s_date['langs'],$data_template['date_features'],'about_','description_',false);
16744: 			fields_for_langs($s_date['langs'],$data_template['date_places'],'','name_',true);
16745: 			fields_for_langs($s_date['langs'],$data_template['date_places'],'about_','description_',false);
16746: 			fields_for_langs($s_date['langs'],$data_template['date_select'],'','name_',true);
16747: 			fields_for_langs($s_date['langs'],$data_template['date_select'],'about_','description_',false);
16748: 			fields_for_langs($s_date['langs'],$data_template['date_select_frequencies'],'','name_',true);
16749: 			fields_for_langs($s_date['langs'],$data_template['date_select_frequencies'],'about_','description_',false);
16750: 			fields_for_langs($s_date['langs'],$data_template['deal_statuses'],'','name_',true);
16751: 			fields_for_langs($s_date['langs'],$data_template['deal_statuses'],'about_','description_',false);
16752: 			fields_for_langs($s_date['langs'],$data_template['drivers_select_sorting'],'','name_',true);
16753: 			fields_for_langs($s_date['langs'],$data_template['drivers_select_sorting'],'about_','description_',false);
16754: 			fields_for_langs($s_date['langs'],$data_template['faqs'],'title_','name_',true);
16755: 			fields_for_langs($s_date['langs'],$data_template['faqs'],'content_','value_',false);
16756: 			fields_for_langs($s_date['langs'],$data_template['faq_groups'],'','name_',true);
16757: 			fields_for_langs($s_date['langs'],$data_template['favorite_addresses'],'','name_',true);
16758: 			fields_for_langs($s_date['langs'],$data_template['favorite_addresses'],'about_','description_',false);
16759: 			fields_for_langs($s_date['langs'],$data_template['field_types'],'','name_',true);
```

### `archive_17012026_1259/taxi/models/api.php:17401`
```php
17391: 					$out['warning']['update cache'] = $res['error'];
17392: 				}
17393: 			}
17394: 
17395: 			return $out;
17396: 		}
17397: 
17398: 		public function selectDataPrivate($data_private = array(), $json_like = '')
17399: 		{
17400: 			if (empty($_SESSION[UID])) {
17401: 				return $this->showError('404', 'error', 'unauthorized access');
17402: 			}
17403: 			if ($this->id_role != 4)
17404: 			{
17405: 				return $this->showError('404', 'error', 'not enough rights');
17406: 			}
17407: 
17408: 			if (empty($json_like))
17409: 			{
17410: 				$data_private_out = $data_private;
17411: 			}
17412: 			else
17413: 			{
17414: 				@$json_like = json_decode($json_like,true);
17415: 				if (empty($json_like) || !is_array($json_like)) 
17416: 				{
17417: 					return $this->showError('404', 'error', 'wrong json_like');
17418: 				}
17419: 				find_arr_like($data_private,$json_like);
17420: 				$data_private_out = $json_like;
17421: 			}
17422: 			
17423: 			if (isset($data_private_out['script_templates']))
17424: 			{
17425: 				foreach($data_private_out['script_templates'] as $id=>$template)
17426: 				{
```

### `archive_17012026_1259/taxi/models/api.php:17444`
```php
17434: 			return array(
17435: 				'code' 		=>	'200',
17436: 				'status' 	=>	'success',		
17437: 				'data' 		=>	$data_private_out
17438: 			);
17439: 		}
17440: 
17441: 		public function selectCart($filter = NULL)
17442: 		{
17443: 			if (empty($_SESSION[UID])) {
17444: 				return $this->showError('404', 'error', 'unauthorized access');
17445: 			}
17446: 			$sql_user = "`id_user` as u_id,";
17447: 			if ($filter == 'all')
17448: 			{
17449: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
17450: 				$sql_where = "1 = 1";
17451: 			}
17452: 			elseif ($filter == 'trip')	
17453: 			{
17454: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter');
17455: 				$sql_where = "`trip`.`driver` = '" . $_SESSION[UID] . "'";
17456: 			}
17457: 			else
17458: 			{
17459: 				$sql_where = "`id_user` = '" . $_SESSION[UID] . "'";
17460: 				$sql_user = "";
17461: 			}
17462: 
17463: 			$s = "SELECT 
17464: 					$sql_user
17465: 					`product` as prod,
17466: 					`ticket`.`id_seat` as prop,
17467: 					`count`,
17468: 					`booking_limit`,
17469: 					`trip`.`from` as sc_id,
```

### `archive_17012026_1259/taxi/models/api.php:17538`
```php
17528: 				unset($d['json']);
17529: 				$d['count'] = (int)$d['count'];
17530: 				add_time_zone($d['booking_limit']);
17531: 				$out['cart'][] = $d;
17532: 			}
17533: 
17534: 			return array(
17535: 				'code' 		=>	'200',
17536: 				'status' 	=>	'success',		
17537: 				'data' 		=>	$out,
17538: 				'auth_user' =>	array(
17539: 									'u_id' => $_SESSION[UID],
17540: 									'u_name' => $_SESSION['name'],
17541: 									'u_family' => $_SESSION['family'],
17542: 									'u_middle' => $_SESSION['middle'],
17543: 									'u_email' => $_SESSION['email'],
17544: 									'u_phone' => $_SESSION['phone'],
17545: 									'u_role' => $_SESSION['id_role'],
17546: 									'u_a_role' => $this->id_role,
17547: 									'u_check_state' => $_SESSION['id_verification_status'],
17548: 									'u_ban' => $_SESSION['user_ban_status'],
17549: 									'u_active' => $_SESSION['active'],
17550: 									'u_photo' => $_SESSION['photo_link'],
17551: 									'u_birthday' => $_SESSION['birthday_date'],
17552: 									'u_lang' => $_SESSION['id_lang'],
17553: 									'u_currency' => $_SESSION['currency'],
17554: 									'u_gps_software' => $_SESSION['id_navigation']
17555: 								)
17556: 			);
17557: 		}
17558: 
17559: 		public function updateCart($product = "", $property = "", $product_count = 1, $item = '')
17560: 		{
17561: 			if (empty($_SESSION[UID])) {
17562: 				return $this->showError('404', 'error', 'unauthorized access');
17563: 			}
```

### `archive_17012026_1259/taxi/models/api.php:17562`
```php
17552: 									'u_lang' => $_SESSION['id_lang'],
17553: 									'u_currency' => $_SESSION['currency'],
17554: 									'u_gps_software' => $_SESSION['id_navigation']
17555: 								)
17556: 			);
17557: 		}
17558: 
17559: 		public function updateCart($product = "", $property = "", $product_count = 1, $item = '')
17560: 		{
17561: 			if (empty($_SESSION[UID])) {
17562: 				return $this->showError('404', 'error', 'unauthorized access');
17563: 			}
17564: 
17565: 			if (strlen($product) === 0) return $this->showError('404', 'error', 'empty prod');
17566: 			if ($property === NULL || strlen($property) == 0) $property = "";
17567: 
17568: 			$q = query("BEGIN");
17569: 			if ($q === false) return $this->showError('404', 'error', 'begin query failed');
17570: 			
17571: 			$sql_delete = "DELETE
17572: 						FROM `cart`
17573: 						WHERE 
17574: 							`id_user` = '" . $_SESSION[UID] . "' AND 
17575: 							`product` = '" . $product . "' AND `property` = '" . $property . "'
17576: 						";
17577: 			switch (DEFAULT_PROFILE)
17578: 			{
17579: 			
17580: 	/*			if (!empty($item))
17581: 				{
17582: 					@$item = json_decode($item,true);
17583: 					if (empty($item) || !is_array($item)) 
17584: 					{
17585: 						return $this->showError('404', 'error', 'wrong item data');
17586: 					}
17587: 					$sql_where = array();
```

### `archive_17012026_1259/taxi/models/api.php:17887`
```php
17877: 			return array(
17878: 				'code' 		=>	'200',
17879: 				'status' 	=>	'success',
17880: 				'data' 		=>	$out
17881: 			);
17882: 		}
17883: 
17884: 		public function queryString($sql = "", $statement = "select", $var = NULL, $query_roles = '', $hash = '')
17885: 		{		
17886: 			if (empty($_SESSION[UID])) {
17887: 				return $this->showError('404', 'error', 'unauthorized access');
17888: 			}			
17889: 
17890: 			if (!empty($query_roles))
17891: 			{
17892: 				if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)
17893: 				{
17894: 					return $this->showError('404', 'error', 'wrong user check state');
17895: 				}
17896: 				$query_roles = explode(',',$query_roles);
17897: 				$query_roles = array_flip($query_roles);
17898: 				if (!isset($query_roles[$this->id_role])) return $this->showError('404', 'error', 'forbidden role');
17899: 			}
17900: 
17901: 			$statement = strtolower($statement);
17902: 			if ($statement !== 'select')
17903: 			{
17904: 				if (in_array($statement,array('update','insert','delete','replace'))){
17905: 					if ($this->constant['query_extended_statements'] === false){
17906: 						return $this->showError('404', 'error', 'statement disabled');
17907: 					}
17908: 				}
17909: 				elseif ($statement == 'custom')
17910: 				{
17911: 					if ($this->id_role != 4)  return $this->showError('404', 'error', 'not enough rights');
17912: 					if ($hash !=  md5('checking' . md5(API_KEY))) return $this->showError('404', 'error', 'wrong hash');
```

### `archive_17012026_1259/taxi/models/api.php:17928`
```php
17918: 				}
17919: 			}
17920: 			if (empty($sql)) return $this->showError('404', 'error', 'empty sql string');
17921: 			$s = trim($sql);
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
```

### `archive_17012026_1259/taxi/models/api.php:17929`
```php
17919: 			}
17920: 			if (empty($sql)) return $this->showError('404', 'error', 'empty sql string');
17921: 			$s = trim($sql);
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
```

### `archive_17012026_1259/taxi/models/api.php:17930`
```php
17920: 			if (empty($sql)) return $this->showError('404', 'error', 'empty sql string');
17921: 			$s = trim($sql);
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
```

### `archive_17012026_1259/taxi/models/api.php:17931`
```php
17921: 			$s = trim($sql);
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
```

### `archive_17012026_1259/taxi/models/api.php:17932`
```php
17922: 			if ($statement != 'custom' && trim(strtolower(substr($s,0,strlen($statement)+1))) !== $statement)
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
```

### `archive_17012026_1259/taxi/models/api.php:17933`
```php
17923: 			{
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
```

### `archive_17012026_1259/taxi/models/api.php:17934`
```php
17924: 				$s = "$statement $s";
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
```

### `archive_17012026_1259/taxi/models/api.php:17935`
```php
17925: 			}
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
```

### `archive_17012026_1259/taxi/models/api.php:17936`
```php
17926: //			mysqli_report(MYSQLI_REPORT_OFF);
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
```

### `archive_17012026_1259/taxi/models/api.php:17937`
```php
17927: 			$sys_data = array(
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
```

### `archive_17012026_1259/taxi/models/api.php:17938`
```php
17928: 								'{$_SYS[AUTH][u_id]}' => $_SESSION[UID],
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
```

### `archive_17012026_1259/taxi/models/api.php:17939`
```php
17929: 								'{$_SYS[AUTH][u_name]}' => $_SESSION['name'],
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
```

### `archive_17012026_1259/taxi/models/api.php:17940`
```php
17930: 								'{$_SYS[AUTH][u_family]}' => $_SESSION['family'],
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
17965: 					$s_arr[] = $s_part;
```

### `archive_17012026_1259/taxi/models/api.php:17941`
```php
17931: 								'{$_SYS[AUTH][u_middle]}' => $_SESSION['middle'],
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
17965: 					$s_arr[] = $s_part;
17966: 					$search_reg_pos += strlen($s_part);
```

### `archive_17012026_1259/taxi/models/api.php:17942`
```php
17932: 								'{$_SYS[AUTH][u_email]}' => $_SESSION['email'] === NULL ? 'NULL' : $_SESSION['email'],
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
17965: 					$s_arr[] = $s_part;
17966: 					$search_reg_pos += strlen($s_part);
17967: 				}
```

### `archive_17012026_1259/taxi/models/api.php:17943`
```php
17933: 								'{$_SYS[AUTH][u_phone]}' => $_SESSION['phone'] === NULL ? 'NULL' : $_SESSION['phone'],
17934: 								'{$_SYS[AUTH][u_tg]}' => $_SESSION['tg'] === NULL ? 'NULL' : $_SESSION['tg'],
17935: 								'{$_SYS[AUTH][u_role]}' => $_SESSION['id_role'],
17936: 								'{$_SYS[AUTH][u_a_role]}' => $this->id_role === NULL ? 'NULL' : $this->id_role,
17937: 								'{$_SYS[AUTH][u_check_state]}' => $_SESSION['id_verification_status'] === NULL ? 'NULL' : $_SESSION['id_verification_status'],
17938: 								'{$_SYS[AUTH][u_ban]}' => empty($_SESSION['user_ban_status']) ? 0 : 1,
17939: 								'{$_SYS[AUTH][u_active]}' => $_SESSION['active'],
17940: 								'{$_SYS[AUTH][u_birthday]}' => $_SESSION['birthday_date'] === NULL ? 'NULL' : $_SESSION['birthday_date'],
17941: 								'{$_SYS[AUTH][u_lang]}' => $_SESSION['id_lang'] === NULL ? 'NULL' : $_SESSION['id_lang'],
17942: 								'{$_SYS[AUTH][lang]}' => $_SESSION['lang'],
17943: 								'{$_SYS[AUTH][u_currency]}' => $_SESSION['currency'] === NULL ? 'NULL' : $_SESSION['currency']
17944: 			);
17945: 			foreach($sys_data as $d_var=>$val)
17946: 			{		
17947: 				$s = str_replace($d_var,real_escape_string($val),$s);
17948: 			}
17949: 
17950: 			if ($var !== NULL && $var !== "")
17951: 			{
17952: 				$q = query("BEGIN");
17953: 				if ($q === false) return $this->showError('404', 'error', error_db());
17954: 			}
17955: 
17956: 			if ($statement == 'custom')
17957: 			{
17958: 				$reg = '/^([^;\'"`#\/]|\'([^\'\\\\]|\\\\.)*\'|"([^"\\\\]|\\\\.)*"|`([^`]|``)+`|#[^\\n\\r]*|\\/\\*([^\/*]|\\*[^\/]|[^*]\\/)*\\*\\/|\\/)+(;\s*|$)/';
17959: 				$search_reg_pos = 0;
17960: 				$s_arr = array();
17961: 				while (true) 
17962: 				{
17963: 					if (!preg_match($reg,substr($s,$search_reg_pos),$reg_res)) break;
17964: 					$s_part = $reg_res[0];
17965: 					$s_arr[] = $s_part;
17966: 					$search_reg_pos += strlen($s_part);
17967: 				}
17968: 				mysqli_report(MYSQLI_REPORT_OFF);
```

### `archive_17012026_1259/taxi/models/api.php:18052`
```php
18042: 			return array(
18043: 				'code' 		=>	'200',
18044: 				'status' 	=>	'success',
18045: 				'data'		=>	$out
18046: 			);
18047: 		}
18048: 
18049: 		public function writeTicket($id_trip = "", $data = "")
18050: 		{
18051: 			if (empty($_SESSION[UID])) {
18052: 				return $this->showError('404', 'error', 'unauthorized access');
18053: 			}
18054: 
18055: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
18056: 
18057: 			if (empty($data)) 
18058: 			{
18059: 				return $this->showError('404', 'error', 'empty data');
18060: 			}
18061: 
18062: 			$data = json_decode($data,true);
18063: 			
18064: 			if (empty($data) || !is_array($data)) 
18065: 			{
18066: 				return $this->showError('404', 'error', 'wrong data');
18067: 			}
18068: 			
18069: 			$filtered_data = array();
18070: 			$seats = array();
18071: 			$folder = $_SERVER['DOCUMENT_ROOT'] . CONFIG_USER_FILE_PATH . "trips/$id_trip/ticket/";
18072: 			foreach($data as $i=>$file_arr)
18073: 			{
18074: 				if(!is_array($file_arr)) return $this->showError('404', 'error', "$i: wrong data");
18075: 				if (isset($file_arr['base64']))
18076: 				{
18077: 					if (empty($file_arr['seat'])) return $this->showError('404', 'error', "$i: empty seat");
```

### `archive_17012026_1259/taxi/models/api.php:18138`
```php
18128: 				LIMIT 1
18129: 				";
18130: 			
18131: 			$q = query($s);
18132: 			if ($q === false) return $this->showError('404', 'error', 'database select failed');
18133: 
18134: 			$d = fetch_assoc($q);
18135: 			if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18136: 			if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18137: 			{
18138: 				return $this->showError('404', 'error', 'user is not author');
18139: 			}
18140: 
18141: 			$t_options = json_decode($d['json'],true);
18142: 			
18143: 			if (empty($t_options['seats_sold']))
18144: 			{
18145: 				if ((int)$d['seats_count'] != count($seats)) return $this->showError('404', 'error', "seat not found");
18146: 			}
18147: 			else
18148: 			{
18149: 				foreach($seats as $seat)
18150: 				{
18151: 					if (!isset($t_options['seats_sold'][$seat])) return $this->showError('404', 'error', "$seat not found");	
18152: 				}
18153: 			}
18154: 			$fix_seats = array();
18155: 			if (!empty($d['fix_seats']))
18156: 			{
18157: 				$fix_seats = explode(chr(1),$d['fix_seats']);
18158: 				$d['fix_seats'] = array();
18159: 				foreach ($fix_seats as $key=>$value)
18160: 				{
18161: 					list(
18162: 						$id_seat,
18163: 						$id_trip_seat
```

### `archive_17012026_1259/taxi/models/api.php:18236`
```php
18226: 			return array(
18227: 				'code' 		=>	'200',
18228: 				'status' 	=>	'success',
18229: 				'data'		=>	$out
18230: 			);
18231: 		}
18232: 
18233: 		public function readTicket($id_trip = "", $seat = "", $pdf = false, $lang_vls = array(), $price_time_functions = array(), $s_data_stt = array(), $aggregators = array())
18234: 		{
18235: 			if (empty($_SESSION[UID])) {
18236: 				return $this->showError('404', 'error', 'unauthorized access');
18237: 			}
18238: 
18239: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
18240: 			
18241: 			if (empty($seat))
18242: 			{
18243: 				$s = "SELECT			
18244: 						`trip`.`id_trip`,
18245: 						`trip`.`driver`,
18246: 						`trip`.`json`,
18247: 						GROUP_CONCAT(`ticket`.`id_seat` SEPARATOR ',') as seats
18248: 					FROM `trip`
18249: 					LEFT JOIN `ticket` ON `ticket`.`id_trip` = `trip`.`id_trip` AND `ticket`.`blob_link` != ''
18250: 					WHERE	
18251: 						`trip`.`id_trip` = '" . $id_trip . "'
18252: 					LIMIT 1
18253: 					";
18254: 				
18255: 				$q = query($s);
18256: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
18257: 
18258: 				$d = fetch_assoc($q);
18259: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18260: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18261: 				{
```

### `archive_17012026_1259/taxi/models/api.php:18262`
```php
18252: 					LIMIT 1
18253: 					";
18254: 				
18255: 				$q = query($s);
18256: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
18257: 
18258: 				$d = fetch_assoc($q);
18259: 				if (empty($d['id_trip'])) return $this->showError('404', 'error', 'trip not found');
18260: 				if ($this->id_role != 4 && $d['driver'] != $_SESSION[UID]) 
18261: 				{
18262: 					return $this->showError('404', 'error', 'user is not author');
18263: 				}
18264: 
18265: 				$t_options = json_decode($d['json'],true);
18266: 				if (empty($t_options['seats_sold']))
18267: 				{
18268: 					$seats = empty($d['seats']) ? array() : explode(',',$d['seats']);
18269: 				}
18270: 				else
18271: 				{
18272: 					$folder = CONFIG_USER_FILE_PATH . "trips/$id_trip/ticket/";
18273: 					$seats = array();				
18274: 					if ($dir = scandir($folder))
18275: 					{
18276: 						foreach($dir as $obj)
18277: 						{
18278: 							if ($obj != "." && $obj != ".." && is_file("$folder$obj"))
18279: 							{
18280: 								$seats[] = $obj;
18281: 							}
18282: 						}
18283: 					}
18284: 				}
18285: 
18286: 				return array(
18287: 					'code' 		=>	'200',
```

### `archive_17012026_1259/taxi/models/api.php:18613`
```php
18603: 					if ($d['blob_mime'] !== NULL && $d['blob_mime'] !== '') header("Content-Type: {$d['blob_mime']}");
18604: 					echo @file_get_contents($path);
18605: 				}
18606: 				exit;
18607: 			}
18608: 		}
18609: 
18610: 		public function selectCartBlock($filter = NULL)
18611: 		{
18612: 			if (empty($_SESSION[UID])) {
18613: 				return $this->showError('404', 'error', 'unauthorized access');
18614: 			}
18615: 
18616: 			$sql_group_concat = "CONCAT_WS(0x00,
18617: 										`cart_block_trip`.`id_trip`,
18618: 										IFNULL(`cart_block_trip`.`status`,0x02),
18619: 										`cart_block_trip`.`status_sys`
18620: 									)";
18621: 			 $sql_left_trip = $sql_left = "";
18622: 			if ($filter == 'all')
18623: 			{
18624: 				if ($this->id_role != 2 && $this->id_role != 4) return $this->showError('404', 'error', 'wrong role for filter all');
18625: 				$sql_field = "`cart_block`.`id_user` as u_id,
18626: 				`cart_block`.`active`,";
18627: 				if ($this->id_role != 4)
18628: 				{
18629: 					$sql_group_concat = "IF(`trip`.`driver` = '{$_SESSION[UID]}',
18630: 								CONCAT_WS(0x00,
18631: 									`cart_block_trip`.`id_trip`,
18632: 									IFNULL(`cart_block_trip`.`status`,0x02),
18633: 									`cart_block_trip`.`status_sys`
18634: 								),NULL)";
18635: 				}
18636: 				$sql_left_trip = "LEFT JOIN `trip` USING (`id_trip`)";
18637: 				$sql_where = "1 = 1";
18638: 			}
```

### `archive_17012026_1259/taxi/models/api.php:18731`
```php
18721: 						}
18722: 					}
18723: 				}
18724: 				$out['cart'][] = $d;
18725: 			}
18726: 
18727: 			return array(
18728: 				'code' 		=>	'200',
18729: 				'status' 	=>	'success',		
18730: 				'data' 		=>	$out,
18731: 				'auth_user' =>	array(
18732: 									'u_id' => $_SESSION[UID],
18733: 									'u_name' => $_SESSION['name'],
18734: 									'u_family' => $_SESSION['family'],
18735: 									'u_middle' => $_SESSION['middle'],
18736: 									'u_email' => $_SESSION['email'],
18737: 									'u_phone' => $_SESSION['phone'],
18738: 									'u_role' => $_SESSION['id_role'],
18739: 									'u_a_role' => $this->id_role,
18740: 									'u_check_state' => $_SESSION['id_verification_status'],
18741: 									'u_ban' => $_SESSION['user_ban_status'],
18742: 									'u_active' => $_SESSION['active'],
18743: 									'u_photo' => $_SESSION['photo_link'],
18744: 									'u_birthday' => $_SESSION['birthday_date'],
18745: 									'u_lang' => $_SESSION['id_lang'],
18746: 									'u_currency' => $_SESSION['currency'],
18747: 									'u_gps_software' => $_SESSION['id_navigation']
18748: 								)
18749: 			);
18750: 		}
18751: 
18752: 		public function updateCartBlock($product = "", $property = "", $product_count = 1, $notice = 1, $data = array(), $data_stt = array())
18753: 		{
18754: 			if (empty($_SESSION[UID])) {
18755: 				return $this->showError('404', 'error', 'unauthorized access');
18756: 			}
```

### `archive_17012026_1259/taxi/models/api.php:18755`
```php
18745: 									'u_lang' => $_SESSION['id_lang'],
18746: 									'u_currency' => $_SESSION['currency'],
18747: 									'u_gps_software' => $_SESSION['id_navigation']
18748: 								)
18749: 			);
18750: 		}
18751: 
18752: 		public function updateCartBlock($product = "", $property = "", $product_count = 1, $notice = 1, $data = array(), $data_stt = array())
18753: 		{
18754: 			if (empty($_SESSION[UID])) {
18755: 				return $this->showError('404', 'error', 'unauthorized access');
18756: 			}
18757: 
18758: 			if (strlen($product) === 0) return $this->showError('404', 'error', 'empty prod');
18759: 			if ($property === NULL) $property = "";
18760: 
18761: 			if ($product_count < 0) $product_count = 0;
18762: 
18763: 			$out = array();
18764: 			if (empty($product_count))
18765: 			{
18766: 				$s = "UPDATE `cart_block`
18767: 					SET
18768: 						`active` = 0
18769: 					WHERE 
18770: 						`id_user` = '" . $_SESSION[UID] . "' AND 
18771: 						`product` = '" . $product . "' AND `property` in ('" . str_replace(';',"','",$property) . "')
18772: 					";
18773: 
18774: 				$q = query($s);
18775: 					
18776: 				if ($q === false) return $this->showError('404', 'error', 'database delete failed');
18777: 			}
18778: 			else
18779: 			{	
18780: 				$sql_property = str_replace(';',"','",$property);
```

### `archive_17012026_1259/taxi/models/api.php:19010`
```php
19000: 			return array(
19001: 				'code' 		=>	'200',
19002: 				'status' 	=>	'success',
19003: 				'data'		=>	$out
19004: 			);	
19005: 		}
19006: 
19007: 		public function sendToTicketBuyerEmail($resend = false, $id_trip = "", $seat = "", $subject = "", $body = "", $file = '', $id_user = "", $u_name = NULL, $email = "", $ignore_order = false, $price = NULL, $lang_vls = array(), $price_time_functions = array(), $s_data_stt = array(), $aggregators = array())
19008: 		{
19009: 			if (empty($_SESSION[UID])) {
19010: 				return $this->showError('404', 'error', 'unauthorized access');
19011: 			}
19012: 
19013: 			if (empty($id_trip)) return $this->showError('404', 'error', 'empty t_id');
19014: 
19015: 			if (empty($seat)) return $this->showError('404', 'error', 'empty seat');
19016: 
19017: 			if ($ignore_order === false)
19018: 			{
19019: 				$id_user = '';
19020: 				$price = NULL;
19021: 			}
19022: 
19023: 			if (!empty($file))
19024: 			{
19025: 				@$file = json_decode($file,true);				
19026: 				if (empty($file) || !is_array($file)) 
19027: 				{
19028: 					return $this->showError('404', 'error', 'wrong file data');
19029: 				}
19030: 			}
19031: 
19032: 			$extended = false;
19033: 			if ($subject == 'template' || $body == 'template')
19034: 			{
19035: 				$extended = true;
```

### `archive_17012026_1259/taxi/models/api.php:19555`
```php
19545: 
19546: 			return array(
19547: 				'code' 		=>	'200',
19548: 				'status' 	=>	'success'
19549: 			);
19550: 		}
19551: 
19552: 		public function clearCart($item = '')
19553: 		{
19554: 			if (empty($_SESSION[UID])) {
19555: 				return $this->showError('404', 'error', 'unauthorized access');
19556: 			}
19557: 
19558: 			if (empty($item))
19559: 			{
19560: 				$sql_where = '';
19561: 			}
19562: 			else		
19563: 			{
19564: 				$sql_where = array();
19565: 				@$item = json_decode($item,true);
19566: 				if (empty($item) || !is_array($item)) 
19567: 				{
19568: 					return $this->showError('404', 'error', 'wrong item data');
19569: 				}
19570: 				foreach($item as $product=>$property)
19571: 				{
19572: 					$str = "(`product` = '$product' AND `property` in ";
19573: 					if (is_array($property))
19574: 					{
19575: 						$property = implode("','",$property);
19576: 					}
19577: 					$property = "(SELECT
19578: 										`id_trip_seat`
19579: 									FROM `ticket`
19580: 									WHERE 
```

### `archive_17012026_1259/taxi/models/api.php:19607`
```php
19597: 
19598: 			return array(
19599: 				'code' 		=>	'200',
19600: 				'status' 	=>	'success'
19601: 			);
19602: 		}
19603: 
19604: 		public function clearCartBlock($item = '')
19605: 		{
19606: 			if (empty($_SESSION[UID])) {
19607: 				return $this->showError('404', 'error', 'unauthorized access');
19608: 			}
19609: 
19610: 			if (empty($item))
19611: 			{
19612: 				$sql_where = '';
19613: 			}
19614: 			else		
19615: 			{
19616: 				$sql_where = array();
19617: 				@$item = json_decode($item,true);
19618: 				if (empty($item) || !is_array($item)) 
19619: 				{
19620: 					return $this->showError('404', 'error', 'wrong item data');
19621: 				}
19622: 				foreach($item as $product=>$property)
19623: 				{
19624: 					$str = "(`product` = '$product' AND `property` in (";
19625: 					if (is_array($property))
19626: 					{
19627: 						$property = implode("','",$property);
19628: 					}
19629: 					$str .= "'$property'))";
19630: 					$sql_where[] = $str;
19631: 				}
19632: 				$sql_where = " AND (" . implode(" OR ",$sql_where) . ")";
```

### `archive_17012026_1259/taxi/models/api.php:19655`
```php
19645: 
19646: 			return array(
19647: 				'code' 		=>	'200',
19648: 				'status' 	=>	'success'
19649: 			);
19650: 		}
19651: 
19652: 		public function startEmailVerification()
19653: 		{
19654: 			if (empty($_SESSION[UID])) {
19655: 				return $this->showError('404', 'error', 'unauthorized access');
19656: 			}
19657: 
19658: 			$s = "SELECT 
19659: 					`email`,
19660: 					`name`,
19661: 					`family`,
19662: 					`middle`,
19663: 					`email_is_verified`
19664: 				FROM `users`
19665: 				WHERE
19666: 					`id_user` = '" . $_SESSION[UID] . "'
19667: 				";		
19668: 
19669: 			$q = query($s);
19670: 			if ($q === false) return $this->showError('404', 'error', 'user select failed');
19671: 
19672: 			$d = fetch_assoc($q);
19673: 			if (empty($d['email'])) return $this->showError('404', 'error', 'empty email');
19674: 			$email = $d['email'];
19675: 			if (!empty($d['email_is_verified'])) return $this->showError('404', 'error', 'verified email');
19676: 
19677: 			preg_match('/^(\+|-)([0-9]{2})\:([0-9]{2})$/',MYSQL_TIME_ZONE,$parsed_time_zone);
19678: 			$start_datetime = gmdate("Y-m-d H:i:s",time() + ($parsed_time_zone[1] === '+' ? 1 : -1)*((int)$parsed_time_zone[2]*60 + (int)$parsed_time_zone[3])*60);
19679: 			$arr = array(
19680: 				'k:id'			=>	$_SESSION[UID],
```

## 5. Exact string matches


Прямого exact-string match между найденными frontend endpoint strings и backend auth contexts не обнаружено.

## 6. Вывод

Цель этого pass — не доказать Auth по тематическому сходству, а найти один конкретный route на обеих сторонах.

Если exact route не сопоставлен, статус остаётся:

```text
UNKNOWN / SOURCE_GAP
```

Наличие auth-related functions на frontend и auth-related PHP на backend не является достаточным доказательством cross-layer relation.

## 7. Следующий шаг

Если exact route найден — открыть только этот frontend function и соответствующий backend handler и проследить:

```text
request fields    ↓backend input    ↓credential creation / identity    ↓response    ↓frontend state    ↓protected request```

Если exact route не найден — остановить эту ветку как SOURCE_GAP и перейти к следующему domain concept, не реконструируя login из косвенных совпадений.