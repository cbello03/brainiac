## Models

### User
- Id: int
- employee_id: str
- firstname: str
- Last name: str
- Username: str
- Password: str
- created_at: timestamp
- updated_at: timestamp

### Role
- Id: int
- Name: str
- created_at: timestamp
- updated_at: timestamp

```javascript
[
  "admin",
  "root",
  "user"
]
```

### User_roles
- user_id: int
- role_id: int

### Permission 
- Id: int
- codename: str 
- description: str
- created_at: timestamp
- updated_at: timestamp

### User permissions
- User_id: int
- Permission_id: int

### Role_Permissions
- role_id: int
- permission_id: int

### Department
- id: int
- name: str
- location: text
- floor: str
- manager_id: int
- parent_department: id
- contact_number: str
- created_at: timestamp
- updated_at: timestamp

### Contact 
- Id: int
- employee_id: str
- job_title: str
- department_id: int
- email: str
- photo_url: text
- bio: text
- is_active: bool
- on_vacation: bool
- user_id: int
- start_date: timestamp
- created_at: timestamp
- updated_at: timestamp

### Contact_number
- id: int
- contact_id: int
- number: str
- created_at: timestamp
- updated_at: timestamp

### Conversation
id: int
type: str // private, group
created_by: int
created_at: timestamp
updated_at: timestamp

### Conversation_Participant
- user_id
- conversation_id
- last_seen_message_id

### Message
- id
- conversation_id
- sender_id
- content
- sent_at
- is_edited

### User_Status
- user_id
- status // online. offline, busy
- updated_at