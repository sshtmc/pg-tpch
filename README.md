
Instructions:

To make this profiler work, you should compile PostgreSQL from source.
I will give you the instructions to do this for the version 9.2 of PostgreSQL,
on Ubuntu 12.04.
You might have to change some steps for other versions of Linux, or for other
versions of PostgreSQL.

Install all dependencies of Postgres
```bash
sudo apt-get build-dep postgresql
```

Install the dependencies of the scripts
```bash
sudo apt-get install graphviz libreadline5-dev zlib1g-dev pgtune
```

Download, build, and install a custom version of Postgres
```bash
wget http://ftp.postgresql.org/pub/source/v9.2.1/postgresql-9.2.1.tar.bz2
tar xf postgresql-9.2.1.tar.bz2
cd postgresql-9.2.1/
CFLAGS="-fno-omit-frame-pointer -rdynamic" ./configure --prefix=$HOME/pg92 --enable-debug
make -j$(grep -c ^processor /proc/cpuinfo)
make install
```

Add Postgres to the path. I assume you are using bash (the default)
```bash
echo -e "\nexport PATH=\$HOME/pg92/bin:\$PATH" >>~/.bashrc
```

Now logout and login. If you don't want to logout at this time, you should run run the following command in all new terminals, until you do logout and login again
```bash
export PATH=$HOME/pg92/bin:$PATH
```

To install the 'perf' profiler and the dependencies
```bash
full_version=$(uname -r)
flavour_abi=${full_version#*-}
flavour=${flavour_abi#*-}
version=${full_version%-$flavour}
sudo apt-get install linux-tools-common linux-tools-${version}
```

For better Postgres performance, you should also consider increasing the default
limits for allocating memory (this increases the limits to 2GB):
```bash
sudo sysctl -w kernel.shmmax=2147483648
sudo sysctl -w kernel.shmall=2097152
```

To make these changes permanent, execute the following:
```bash
cat <<__EOF | sudo tee -a /etc/sysctl.conf
kernel.shmmax=2147483648
kernel.shmall=2097152
__EOF
```

You are now ready to create and populate the TPC-H database and the tables:
```bash
./tpch_prepare
```

And after it finishes, run the queries and generate the call graphs:
```bash
./tpch_runall_seq
```
This will create a directory perfdata, and put all the evaluation results in
there.

You may also use a custom directory for the performance results, by giving an
argument to the tpch_runall_seq:
```bash
./tpch_runall_seq my-config
```
This will create a directory perfdata-my-config and put the results in it.

