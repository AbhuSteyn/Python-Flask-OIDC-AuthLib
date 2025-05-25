
## **OIDC Token Validation in Flask Using Authlib**
Authlib is a **lightweight OAuth/OIDC library** for Python that handles **token validation, introspection, and user authentication seamlessly**, making the implementation much easier compared to manually handling JWT verification.

---

### **Step 1: Install Dependencies**
```bash
pip install flask authlib
```

---

### **Step 2: Configure Flask for OIDC Authentication**
Create `app.py`:
```python
from flask import Flask, request, jsonify
from authlib.integrations.flask_client import OAuth

app = Flask(__name__)
app.config["OIDC_CLIENT_ID"] = "your-client-id"
app.config["OIDC_CLIENT_SECRET"] = "your-client-secret"
app.config["OIDC_DISCOVERY_URL"] = "https://your-oidc-provider.com/.well-known/openid-configuration"

# Initialize OAuth client
oauth = OAuth(app)
oidc = oauth.register(
    name="oidc",
    client_id=app.config["OIDC_CLIENT_ID"],
    client_secret=app.config["OIDC_CLIENT_SECRET"],
    server_metadata_url=app.config["OIDC_DISCOVERY_URL"],
    client_kwargs={"scope": "openid profile email"}
)

# Protected route
@app.route("/secure-data")
def secure_data():
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    if not token:
        return jsonify({"error": "Missing token"}), 401

    try:
        user_info = oidc.parse_id_token(token)  # Automatically validates the token
        return jsonify({"message": "Secure Data Accessed", "user": user_info})
    except Exception:
        return jsonify({"error": "Invalid or expired token"}), 401

if __name__ == "__main__":
    app.run(debug=True)
```

---

### **Step 3: Running the Flask API**
```bash
python app.py
```

---

### **Step 4: Testing API with OIDC Token**
#### **Make an API Request with a Valid JWT**
```bash
curl -X GET -H "Authorization: Bearer <ACCESS_TOKEN>" http://127.0.0.1:5000/secure-data
```
Expected Response:
```json
{
    "message": "Secure Data Accessed",
    "user": { "sub": "user-id", "email": "user@example.com" }
}
```

#### **Call API with an Invalid Token**
```bash
curl -X GET -H "Authorization: Bearer invalid_token" http://127.0.0.1:5000/secure-data
```
Expected Response:
```json
{
    "error": "Invalid or expired token"
}
```

---

### **Why Is This Better Than Manual JWT Validation?**
✅ **Automatic Token Parsing & Validation**  
✅ **No Manual JWKS Handling**  
✅ **Easier Configuration & OIDC Discovery Support**  
✅ **Works with Providers like Azure AD, Auth0, Okta, Keycloak**  
