Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

```yaml
organizations:  # List of organizations to be configured in Foreman and AWX
  - name: Default  # Name of the organizations, there must be one Default organization
    usergroups:  # List of user groups under this organization
      - name: administrators  # Name of the internal user group
        admin: true  # Flag to indicate this user group have administrative rights
        ldap_cn: admin  # Optional: LDAP common name of users in this group, also the name of the internal user group
      - name: managers
        roles:  # List of roles assigned to users in this group
          - Manager
        ldap_cn: users
    locations:  # List of locations under this organization
      - name: Default  # The Default organization must contain a Default location
      - name: Warehouse A
      - name: ITC
        
hostgroups:  # Defines hierarchical host group relationships  
  - name: all
    description: Parent of all the groups
    children:  # Recursive list of child hostgroups in the parent
      - name: foreman
        description: Builds and configures hosts
      - name: awx
        description: Bare metal hosts

domains:  # List of domains available to Foreman and AWX
  - name: dev.home.prettybaked.com
    description: Development Domain
  - name: home.prettybaked.com
    description: Production Domain

subnets:  # List of subnet specifications
  - name: 10.0.10.0/24  # Subnet name, recommended to be the CIDR
    description: PROD VLAN 10  # Description of the subnet
    bridge_interface: br10  # The host interface name bridging servers to this subnet
    gateway: 10.0.10.1  # The IPv4 gateway address
    boot_mode: DHCP  # Defines the boot mode for servers on this subnet
    vlan_id: 10  # The VLAN identifier this subnet is on
    domains:  # List of domains associated with this subnet, items must match the name attribute in the domains variable
      - home.prettybaked.com
      - dev.home.pretybaked.com
    tftp_proxy: "smart-proxy"  # Optional: Proxy to use for TFTP
    remote_execution_proxies: "smart-proxy"  # Optional: Proxy to use for Remote Execution
    discovery_proxy: "smart-proxy"  # Optional: Proxy to use for Discovery
    httpboot_proxy: "smart-proxy"  # Optional: Proxy to use for HTTP boot
  - name: 10.0.100.0/24
    description: Development subnet representation of VLAN 10
    bridge_interface: br0
    gateway: 10.0.100.1
    boot_mode: DHCP
    domains:
      - dev.home.prettybaked.com
    tftp_proxy: "smart-proxy"
    remote_execution_proxies: "smart-proxy"
    discovery_proxy: "smart-proxy"
    httpboot_proxy: "smart-proxy"

default_proxies_enabled:  # List of proxies to be enabled by default
  - tftp
  - remote_execution
  - discovery
  - httpboot
    
foreman_options:  # List of options to pass to the installer
  - enable-foreman-cli-discovery
  - enable-foreman-compute-libvirt
  - enable-foreman-plugin-discovery
  - enable-foreman-proxy-plugin-discovery
  - foreman-proxy-plugin-discovery-install-images=true
  - enable-foreman-plugin-remote-execution
  - enable-foreman-proxy
  - foreman-proxy-tftp=true
  - "foreman-proxy-tftp-root={{ foreman_tftp_root }}"
  - foreman-proxy-httpboot=true
  - foreman-proxy-httpboot-listen-on=http
  - 'foreman-foreman-url={{ foreman.url }}'
  - foreman-websockets-encrypt=false
  - foreman-proxy-bind-host=0.0.0.0
  - foreman-proxy-ssl=false
  - "foreman-proxy-foreman-base-url=http://10.0.10.155"
  - 'foreman-initial-admin-username={{ foreman.admin_username }}'
  - 'foreman-initial-admin-password={{ foreman.admin_password }}'

foreman:  # Define the Foreman server attributes
  url: 'https://10.0.10.155'  # Address to reach the Foreman instance
  validate_certs: false  # Flag for whether SSL certifications should be validated
  admin_username: !vault |  # The username assigned to Foreman's root admin user
          $ANSIBLE_VAULT;1.1;AES256
          61306132376636316637326464616438363638353565333836633561303132323364643531393066
          6461613539373333366663353339313364343030623564650a323238343064376634636561626232
          38376239363164653661323066303333333932343131393862303934353433376138613132663762
          3233396136623838390a346133666631613635383961316266393763613332663765353538383361
          3765
  admin_password: !vault |  # The password assigned to Foreman's root admin user
          $ANSIBLE_VAULT;1.1;AES256
          33643032623362303131663736333066346163626536306361653062613236656136393664313430
          3534326365623334636561366366383232326432613731370a623631633831336165393730646463
          39623039383837363134363231623763616362316563396632386463343534343835353363626661
          3837663239373561390a353564393639373038316536306530643038663265333130373139373239
          33626639306564613839666131363432336365636336383539613536643163373430
  bmc_password: !vault |  # The BMC password allowing Foreman to manage servers
          $ANSIBLE_VAULT;1.1;AES256
          37646631366639373332326162636538346363363231623434656233663831623165316639343837
          3536353964663433383665326565363462393066373737360a366431323133613831366462326530
          31623165663665393730383837353039313234653437306636316337306331656561663438613964
          6131363161326266370a323930323531323532613339303237323265643137356438356462356430
          6663

awx:  # Define the AWX server attributes
  url: 'http://10.0.10.155'
  port: 8080  # The port AWX web is listening on
  validate_certs: false  # Flag for whether SSL certifications should be validated
  admin_user: !vault |  # The username assigned to AWX's root admin user
          $ANSIBLE_VAULT;1.1;AES256
          36306333343663313037333564373966633238646231626538626530353530616237633930393331
          6639636232303830613661353932633231373161633839640a326234666230303934376230323464
          38613330336238306664323965333761643761353039633338626362666238646331316239313364
          3330666431343731300a376339383564656637343033353764663333616137333966313330663239
          3334
  admin_password: !vault |  # The password assigned to AWX's root admin user
          $ANSIBLE_VAULT;1.1;AES256
          66343636316534613935306539376363373063616535346330613636383163353435336233363463
          3538383735373262356137663566323938363764616336380a653537303866643637333132356131
          62393062356563323464346430326166643965636530626264356564643038653563393835626438
          3934313434633762340a376566326635333931623332356361346339613464313561393437386130
          3636
awx_docker_compose_dir: /var/lib/awxcompose  # Path on host to AWX docker-compose shared files
awx_postgres_data_dir: /var/lib/awxpostgres  # Path on host to AWX postgres data files
awx_project_data_dir: /etc/ansible  # Path on host to AWX projects

ldap_server: ldap.home.prettybaked.com   # Host for LDAP authentication

ldap_authentication:  # Defines properties required to implement LDAP authentication
  host: 10.0.10.15  # IP address of LDAP server
  server: {{ ldap_server }}
  server_port: 389  # Port LDAP server listens on
  server_tls: false  # Flag for if LDAP uses TLS protocol
  account_dn: 'cn=Manager,{{ ldap_base_dn }}'  # The distinguished name for management account in LDAP
  account_password: !vault |  # The management account password in LDAP
          $ANSIBLE_VAULT;1.1;AES256
          32326562353861373434393865323461393836333265666138656664353235643630646461613835
          3336623364303931653339633933346162643733313931660a316561386437643230623636373833
          63383834616230353932643366356639343361623337366533306566353630616430323365313864
          6133663831656161330a653762626463353132393736643731363930373465663032363635373630
          65316164666132326464383233346365633662653335663163663066636638383237666437316235
          3336353165313736666431306539643061623639666263363264
  base_dn: 'dc={{ ",dc=".join( ldap_server.split(".")) }}'  # LDAP base DN
  groups_base: 'ou=groups,dc={{ ",dc=".join( ldap_server.split(".")) }}'  # LDAP organizational unit for groups
  users_base: 'ou=users,dc={{ ",dc=".join( ldap_server.split(".")) }}'  # LDAP organizational unit for users
  attr_login: uid  # User identifier LDAP attribute 
  attr_firstname: givenName  # User firstname LDAP attribute
  attr_lastname: sn  # User lastname LDAP attribute
  attr_mail: mail  # User email LDAP attribute
  attr_photo: jpegPhoto  # User profile image LDAP attribute
```

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: buildserver
      tasks:
        - ansible.builtin.include_role
            name: ansible-foreman-awx

License
-------

MIT

Author Information
------------------

Teagan Glenn (that@teagantotally.rocks)
