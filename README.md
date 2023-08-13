# Active-Active
```
vrrp_script check_haproxy {
   script "sh /usr/local/bin/haproxy-service-check.sh"
   interval 2
   fall 2
   rise 2
   weight 2
}

global_defs {
   notification_email {
      {{ keepalived.email_notif }}
   }
}

vrrp_instance LB1 {
   state MASTER
   interface {{ keepalived.interface }}
   virtual_router_id {{ keepalived.virtual_router_id_1 }}
   priority {{ keepalived.priority1_1 }}
   advert_int {{ keepalived.advert_int }}
   accept

   authentication {
      auth_type PASS
      auth_pass {{ keepalived_auth_pass }}
   }

   virtual_ipaddress {
      {{ keepalived.vip1_1 }} dev {{ keepalived.interface }}
      {{ keepalived.vip1_2 }} dev {{ keepalived.interface }}
   }

   track_script {
      check_haproxy
   }
   
   track_interface {
      {{ keepalived.interface }}
   }

}

vrrp_instance LB2 {
   state BACKUP
   interface {{ keepalived.interface }}
   virtual_router_id {{ keepalived.virtual_router_id_2 }}
   priority {{ keepalived.priority2_1 }}
   advert_int {{ keepalived.advert_int }}
   accept

   authentication {
      auth_type PASS
      auth_pass {{ keepalived_auth_pass }}
   }

   virtual_ipaddress {
      {{ keepalived.vip2_1 }} dev {{ keepalived.interface }}
      {{ keepalived.vip2_2 }} dev {{ keepalived.interface }}
   }

   track_script {
      check_haproxy
   }
   
   track_interface {
      {{ keepalived.interface }}
   }

}
```
# Active-Passive
# for alternative:
```
global_defs {
   enable_script_security
   script_user root
}
# Script used to check if HAProxy is running
vrrp_script check_haproxy {
   script "killall -0 haproxy"
   interval 2
   weight 2
}

vrrp_instance KUBE_API_LB {
   state BACKUP
   interface ens160
   virtual_router_id 51
   priority 100
   virtual_ipaddress {
      ${vip_api}/32
   }
   track_script {
      check_haproxy
   }
}
```
# for master:
```
global_defs {
   enable_script_security
   script_user root
}

vrrp_script check_haproxy {
   script "killall -0 haproxy"
   interval 2
   weight 2
   }

vrrp_instance KUBE_API_LB {
   state MASTER
   interface ens160
   virtual_router_id 51
   priority 101
   # The virtual ip address shared between the two loadbalancers
   virtual_ipaddress {
      ${vip_api}/32
   }
   track_script {
      check_haproxy
   }
}
```


