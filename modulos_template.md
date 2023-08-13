### Módulo 'user'

	- name: Add the user 'johnd' with a specific uid and a primary group of 'admin'
	  ansible.builtin.user:
		name: johnd
		comment: John Doe
		uid: 1040
		group: admin

	- name: Add the user 'james' with a bash shell, appending the group 'admins' and 'developers' to the user's groups
	  ansible.builtin.user:
		name: james
		shell: /bin/bash
		groups: admins,developers
		append: yes

	- name: Remove the user 'johnd'
	  ansible.builtin.user:
		name: johnd
		state: absent
		remove: yes

	- name: Create a 2048-bit SSH key for user jsmith in ~jsmith/.ssh/id_rsa
	  ansible.builtin.user:
		name: jsmith
		generate_ssh_key: yes
		ssh_key_bits: 2048
		ssh_key_file: .ssh/id_rsa

	- name: Added a consultant whose account you want to expire
	  ansible.builtin.user:
		name: james18
		shell: /bin/zsh
		groups: developers
		expires: 1422403387

	- name: Starting at Ansible 2.6, modify user, remove expiry time
	  ansible.builtin.user:
		name: james18
		expires: -1

	- name: Set maximum expiration date for password
	  ansible.builtin.user:
		name: ram19
		password_expire_max: 10

	- name: Set minimum expiration date for password
	  ansible.builtin.user:
		name: pushkar15
		password_expire_min: 5

---

### Módulo 'lineinfile'

	# NOTE: Before 2.3, option 'dest', 'destfile' or 'name' was used instead of 'path'
	- name: Ensure SELinux is set to enforcing mode
	  ansible.builtin.lineinfile:
		path: /etc/selinux/config
		regexp: '^SELINUX='
		line: SELINUX=enforcing

	- name: Make sure group wheel is not in the sudoers configuration
	  ansible.builtin.lineinfile:
		path: /etc/sudoers
		state: absent
		regexp: '^%wheel'

	- name: Replace a localhost entry with our own
	  ansible.builtin.lineinfile:
		path: /etc/hosts
		regexp: '^127\.0\.0\.1'
		line: 127.0.0.1 localhost
		owner: root
		group: root
		mode: '0644'

	- name: Replace a localhost entry searching for a literal string to avoid escaping
	  ansible.builtin.lineinfile:
		path: /etc/hosts
		search_string: '127.0.0.1'
		line: 127.0.0.1 localhost
		owner: root
		group: root
		mode: '0644'

	- name: Ensure the default Apache port is 8080
	  ansible.builtin.lineinfile:
		path: /etc/httpd/conf/httpd.conf
		regexp: '^Listen '
		insertafter: '^#Listen '
		line: Listen 8080

	- name: Ensure php extension matches new pattern
	  ansible.builtin.lineinfile:
		path: /etc/httpd/conf/httpd.conf
		search_string: '<FilesMatch ".php[45]?$">'
		insertafter: '^\t<Location \/>\n'
		line: '        <FilesMatch ".php[34]?$">'

	- name: Ensure we have our own comment added to /etc/services
	  ansible.builtin.lineinfile:
		path: /etc/services
		regexp: '^# port for http'
		insertbefore: '^www.*80/tcp'
		line: '# port for http by default'

	- name: Add a line to a file if the file does not exist, without passing regexp
	  ansible.builtin.lineinfile:
		path: /tmp/testfile
		line: 192.168.1.99 foo.lab.net foo
		create: yes

	# NOTE: Yaml requires escaping backslashes in double quotes but not in single quotes
	- name: Ensure the JBoss memory settings are exactly as needed
	  ansible.builtin.lineinfile:
		path: /opt/jboss-as/bin/standalone.conf
		regexp: '^(.*)Xms(\d+)m(.*)$'
		line: '\1Xms${xms}m\3'
		backrefs: yes

	# NOTE: Fully quoted because of the ': ' on the line. See the Gotchas in the YAML docs.
	- name: Validate the sudoers file before saving
	  ansible.builtin.lineinfile:
		path: /etc/sudoers
		state: present
		regexp: '^%ADMIN ALL='
		line: '%ADMIN ALL=(ALL) NOPASSWD: ALL'
		validate: /usr/sbin/visudo -cf %s

	# See https://docs.python.org/3/library/re.html for further details on syntax
	- name: Use backrefs with alternative group syntax to avoid conflicts with variable values
	  ansible.builtin.lineinfile:
		path: /tmp/config
		regexp: ^(host=).*
		line: \g<1>{{ hostname }}
		backrefs: yes

---

### Módulo 'file'

	- name: Change file ownership, group and permissions
	  ansible.builtin.file:
		path: /etc/foo.conf
		owner: foo
		group: foo
		mode: '0644'

	- name: Give insecure permissions to an existing file
	  ansible.builtin.file:
		path: /work
		owner: root
		group: root
		mode: '1777'

	- name: Create a symbolic link
	  ansible.builtin.file:
		src: /file/to/link/to
		dest: /path/to/symlink
		owner: foo
		group: foo
		state: link

	- name: Create two hard links
	  ansible.builtin.file:
		src: '/tmp/{{ item.src }}'
		dest: '{{ item.dest }}'
		state: hard
	  loop:
		- { src: x, dest: y }
		- { src: z, dest: k }

	- name: Touch a file, using symbolic modes to set the permissions (equivalent to 0644)
	  ansible.builtin.file:
		path: /etc/foo.conf
		state: touch
		mode: u=rw,g=r,o=r

	- name: Touch the same file, but add/remove some permissions
	  ansible.builtin.file:
		path: /etc/foo.conf
		state: touch
		mode: u+rw,g-wx,o-rwx

	- name: Touch again the same file, but do not change times this makes the task idempotent
	  ansible.builtin.file:
		path: /etc/foo.conf
		state: touch
		mode: u+rw,g-wx,o-rwx
		modification_time: preserve
		access_time: preserve

	- name: Create a directory if it does not exist
	  ansible.builtin.file:
		path: /etc/some_directory
		state: directory
		mode: '0755'

	- name: Update modification and access time of given file
	  ansible.builtin.file:
		path: /etc/some_file
		state: file
		modification_time: now
		access_time: now

	- name: Set access time based on seconds from epoch value
	  ansible.builtin.file:
		path: /etc/another_file
		state: file
		access_time: '{{ "%Y%m%d%H%M.%S" | strftime(stat_var.stat.atime) }}'

	- name: Recursively change ownership of a directory
	  ansible.builtin.file:
		path: /etc/foo
		state: directory
		recurse: yes
		owner: foo
		group: foo

	- name: Remove file (delete file)
	  ansible.builtin.file:
		path: /etc/foo.txt
		state: absent

	- name: Recursively remove directory
	  ansible.builtin.file:
		path: /etc/foo
		state: absent

---

### Módulo 'copy'

    - name: Copy file with owner and permissions
      ansible.builtin.copy:
        src: /srv/myfiles/foo.conf
        dest: /etc/foo.conf
        owner: foo
        group: foo
        mode: '0644'

    - name: Copy file with owner and permission, using symbolic representation
      ansible.builtin.copy:
        src: /srv/myfiles/foo.conf
        dest: /etc/foo.conf
        owner: foo
        group: foo
        mode: u=rw,g=r,o=r

    - name: Another symbolic mode example, adding some permissions and removing others
      ansible.builtin.copy:
        src: /srv/myfiles/foo.conf
        dest: /etc/foo.conf
        owner: foo
        group: foo
        mode: u+rw,g-wx,o-rwx

    - name: Copy a new "ntp.conf" file into place, backing up the original if it differs from the copied version
      ansible.builtin.copy:
        src: /mine/ntp.conf
        dest: /etc/ntp.conf
        owner: root
        group: root
        mode: '0644'
        backup: yes

    - name: Copy a new "sudoers" file into place, after passing validation with visudo
      ansible.builtin.copy:
        src: /mine/sudoers
        dest: /etc/sudoers
        validate: /usr/sbin/visudo -csf %s

    - name: Copy a "sudoers" file on the remote machine for editing
      ansible.builtin.copy:
        src: /etc/sudoers
        dest: /etc/sudoers.edit
        remote_src: yes
        validate: /usr/sbin/visudo -csf %s

    - name: Copy using inline content
      ansible.builtin.copy:
        content: '# This file was moved to /etc/other.conf'
        dest: /etc/mine.conf

    - name: If follow=yes, /path/to/file will be overwritten by contents of foo.conf
      ansible.builtin.copy:
        src: /etc/foo.conf
        dest: /path/to/link  # link to /path/to/file
        follow: yes

    - name: If follow=no, /path/to/link will become a file and be overwritten by contents of foo.conf
      ansible.builtin.copy:
        src: /etc/foo.conf
        dest: /path/to/link  # link to /path/to/file
        follow: no

---

### Módulo 'apt'

    - name: Install apache httpd  (state=present is optional)
      ansible.builtin.apt:
        name: apache2
        state: present

    - name: Update repositories cache and install "foo" package
      ansible.builtin.apt:
        name: foo
        update_cache: yes

    - name: Remove "foo" package
      ansible.builtin.apt:
        name: foo
        state: absent

    - name: Install the package "foo"
      ansible.builtin.apt:
        name: foo

    - name: Install a list of packages
      ansible.builtin.apt:
        pkg:
        - foo
        - foo-tools

    - name: Install the version '1.00' of package "foo"
      ansible.builtin.apt:
        name: foo=1.00

    - name: Update the repository cache and update package "nginx" to latest version using default release squeeze-backport
      ansible.builtin.apt:
        name: nginx
        state: latest
        default_release: squeeze-backports
        update_cache: yes

    - name: Install the version '1.18.0' of package "nginx" and allow potential downgrades
      ansible.builtin.apt:
        name: nginx=1.18.0
        state: present
        allow_downgrade: yes

    - name: Install zfsutils-linux with ensuring conflicted packages (e.g. zfs-fuse) will not be removed.
      ansible.builtin.apt:
        name: zfsutils-linux
        state: latest
        fail_on_autoremove: yes

    - name: Install latest version of "openjdk-6-jdk" ignoring "install-recommends"
      ansible.builtin.apt:
        name: openjdk-6-jdk
        state: latest
        install_recommends: no

    - name: Update all packages to their latest version
      ansible.builtin.apt:
        name: "*"
        state: latest

    - name: Upgrade the OS (apt-get dist-upgrade)
      ansible.builtin.apt:
        upgrade: dist

    - name: Run the equivalent of "apt-get update" as a separate step
      ansible.builtin.apt:
        update_cache: yes

    - name: Only run "update_cache=yes" if the last one is more than 3600 seconds ago
      ansible.builtin.apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Pass options to dpkg on run
      ansible.builtin.apt:
        upgrade: dist
        update_cache: yes
        dpkg_options: 'force-confold,force-confdef'

    - name: Install a .deb package
      ansible.builtin.apt:
        deb: /tmp/mypackage.deb

    - name: Install the build dependencies for package "foo"
      ansible.builtin.apt:
        pkg: foo
        state: build-dep

    - name: Install a .deb package from the internet
      ansible.builtin.apt:
        deb: https://example.com/python-ppq_0.1-1_all.deb

    - name: Remove useless packages from the cache
      ansible.builtin.apt:
        autoclean: yes

    - name: Remove dependencies that are no longer required
      ansible.builtin.apt:
        autoremove: yes

    - name: Run the equivalent of "apt-get clean" as a separate step
      apt:
        clean: yes
