## Install pgvecto.rs PostgreSQL module

### Install from release


Download the deb package in the release page v0.1.11

~~~bash
wget https://github.com/tensorchord/pgvecto.rs/releases/download/v0.1.11/vectors-pg14-v0.1.11-x86_64-unknown-linux-gnu.deb
~~~

and install the package:

~~~bash
sudo dpkg -i vectors-pg14-v0.1.11-x86_64-unknown-linux-gnu.deb
~~~

Configure your PostgreSQL by modifying the shared_preload_libraries to include vectors.so.

~~~bash
psql -U postgres -c 'ALTER SYSTEM SET shared_preload_libraries = "vectors.so"'
~~~

You need restart the PostgreSQL cluster.

~~~bash
sudo systemctl restart postgresql.service
~~~

Connect to the database and enable the extension.

~~~bash
DROP EXTENSION IF EXISTS vectors;
CREATE EXTENSION vectors;
CREATE EXTENSION cube;
CREATE EXTENSION earthdistance;
~~~

~~~bash
sudo systemctl restart postgresql.service
~~~


### Install from source


Install base dependency.

~~~bash
sudo apt install -y \
    build-essential \
    libpq-dev \
    libssl-dev \
    pkg-config \
    gcc \
    libreadline-dev \
    flex \
    bison \
    libxml2-dev \
    libxslt-dev \
    libxml2-utils \
    xsltproc \
    zlib1g-dev \
    ccache \
    clang \
    git
~~~


Install Rust. The following command will install Rustup, the Rust toolchain installer for your user. Do not install rustc using package manager.

~~~bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~~

Install PostgreSQL and its headers. We assume you may install PostgreSQL 15. Feel free to replace 15 to any other major version number you need.

~~~bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install libpq-dev postgresql-15 postgresql-server-dev-15
~~~


Install clang-16. We do not support other versions of clang.

~~~bash
sudo sh -c 'echo "deb http://apt.llvm.org/$(lsb_release -cs)/ llvm-toolchain-$(lsb_release -cs)-16 main" >> /etc/apt/sources.list'
wget --quiet -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install clang-16
~~~

Clone the Repository. Note the following commands are executed in the cloned repository directory.

~~~bash
git clone https://github.com/tensorchord/pgvecto.rs.git
cd pgvecto.rs
~~~

Install cargo-pgrx.

~~~bash 
cargo install cargo-pgrx@$(grep 'pgrx = {' Cargo.toml | cut -d '"' -f 2)
cargo pgrx init --pg15=/usr/lib/postgresql/15/bin/pg_config
~~~

Install pgvecto.rs.

~~~bash
cargo pgrx install --sudo --release
~~~

Configure your PostgreSQL by modifying the shared_preload_libraries to include vectors.so.

~~~bash
psql -U postgres -c 'ALTER SYSTEM SET shared_preload_libraries = "vectors.so"'
~~~

You need restart the PostgreSQL cluster.

~~~bash
sudo systemctl restart postgresql.service
~~~

Connect to the database and enable the extension.

~~~bash
DROP EXTENSION IF EXISTS vectors;
CREATE EXTENSION vectors;
~~~

