<!DOCTYPE html>
<html lang="en">
<body>

<h1>Discourse Installation Guide (Debian 12, Root User)</h1>

<p>This guide explains how to install Discourse on Debian 12 using the <strong>root user</strong>.</p>

<h2>Important Notice (Security Risks):</h2>

<ul>
    <li><strong>Running all commands directly as the root user poses security risks.</strong> If security vulnerabilities exist in Discourse or Docker, attackers may more easily gain root access to your system.</li>
    <li>It is <strong>strongly recommended</strong> to create a regular user and use <code>sudo</code> to execute commands that require root privileges. Please refer to other more secure installation tutorials.</li>
    <li>Only proceed with running all commands directly as the root user if you <em>fully understand</em> the security risks and have a valid reason for doing so.</li>
</ul>

<h2>Installation Steps</h2>

<h3>1. System Preparation</h3>

<ul>
    <li>Connect to your VPS via SSH and log in as the root user.</li>
    <li>Update the system:
        <pre><code>apt update
apt upgrade -y</code></pre>
    </li>
    <li>Install <code>sudo</code> (if not already installed, although subsequent commands may use it):
        <pre><code>apt install -y sudo</code></pre>
    </li>
</ul>

<h3>2. Install Docker</h3>

<ul>
    <li>Install dependencies:
        <pre><code>apt update
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release</code></pre>
    </li>
    <li>Add Docker's official GPG key:
        <pre><code>curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg</code></pre>
    </li>
    <li>Set up the Docker stable repository:
        <pre><code>echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null</code></pre>
    </li>
    <li>Install Docker Engine, containerd, and Docker Compose plugin:
        <pre><code>apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin</code></pre>
    </li>
    <li>Verify Docker installation:
        <pre><code>docker run hello-world</code></pre>
    </li>
</ul>

<h3>3. Install Git</h3>

<pre><code>apt update
apt install -y git</code></pre>

<h3>4. Configure Docker Logging (Optional, but Recommended)</h3>

<pre><code>nano /etc/docker/daemon.json</code></pre>
<ul>
    <li>Copy the following content into the file:
        <pre><code>{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "50m",
        "max-file": "5"
    }
}</code></pre>
    </li>
    <li>Save and exit (Ctrl+X, Y, Enter).</li>
    <li>Restart Docker:
        <pre><code>systemctl restart docker</code></pre>
    </li>
</ul>

<h3>5. Install Discourse</h3>

<ul>
    <li>Create the Discourse installation directory:
        <pre><code>mkdir -p /var/discourse
cd /var/discourse</code></pre>
    </li>
    <li>Clone the Discourse Docker repository:
        <pre><code>git clone https://github.com/discourse/discourse_docker.git .</code></pre>
    </li>
    <li>Copy and edit the configuration file:
        <pre><code>cp samples/standalone.yml containers/app.yml
nano containers/app.yml</code></pre>
    </li>
    <li>Modify the <code>app.yml</code> configuration file (Important):
        <pre><code>DISCOURSE_HOSTNAME: your forum domain (e.g., forum.example.com).

DISCOURSE_DEVELOPER_EMAILS: administrator email addresses (comma-separated for multiple addresses).

DISCOURSE_SMTP_ADDRESS: SMTP server address.

DISCOURSE_SMTP_PORT: SMTP server port.

DISCOURSE_SMTP_USER_NAME: SMTP username.

DISCOURSE_SMTP_PASSWORD: SMTP password.

db_shared_buffers: adjust according to your memory size; for 2.5GB RAM, it is recommended to set to 683MB.

UNICORN_WORKERS: adjust according to your CPU core count; for a 2-core CPU, it is recommended to set to 2.</code></pre>
    </li>
    <li>(Optional) Add the automatic skip email verification registration plugin (e.g., discourse-auth-no-email-verification):
        <pre><code>            - git clone https://github.com/thenogodcom/discourse-auth-no-email-verification.git</code></pre>
    </li>
    <li>Save and close the <code>app.yml</code> file.</li>
    <li>Bootstrap the Discourse container:
        <pre><code>./launcher bootstrap app</code></pre>
    </li>
    <li>Launch the Discourse container:
        <pre><code>./launcher start app</code></pre>
    </li>
</ul>

<h3>6. Complete Discourse Setup Wizard</h3>

<ul>
    <li>Access your forum domain (the <code>DISCOURSE_HOSTNAME</code> you set in <code>app.yml</code>) in a web browser.</li>
    <li>Follow the prompts in the Discourse setup wizard to complete the initial configuration.</li>
    <li>If you chose to add the automatic skip email verification registration plugin, after registration, visit:
        <pre><code>https://DISCOURSE_HOSTNAME/t/welcome-to-discourse/5</code></pre>
    </li>
</ul>

<p><strong>Again, it is emphasized that using the root user to execute all commands poses security risks. Please operate with caution.</strong></p>

</body>
</html>
