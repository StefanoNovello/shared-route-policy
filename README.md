# shared-route-policy
An NSO service package to manage a route-policy that is being used by many service instances. The package exposes 2 services
basic-shared-policy which manages the underlying policy by concatenating chunks of policy

shared-route-policy whose template is in terms of basic-route-policy and whose intent is to add a if-match-set statement into a speciffic route policy.

Tested on IOS-XR

This is the first service created.
```
admin@ncs(config)# shared-route-policy alwaysoniosxr 11:11 set 22:22
admin@ncs(config-shared-route-policy-alwaysoniosxr/11:11)# commit dry-run outformat native
native {
    device {
        name alwaysoniosxr
        data route-policy VRF-shared-1-EXP-RPL
               if community matches-any (11:11) then
                 set extcommunity rt (22:22) additive
               endif
              end-policy
             !
    }
}
admin@ncs(config-shared-route-policy-alwaysoniosxr/11:11)# commit
Commit complete.
```
Then if we add a second service we see how the policy is concatenated, and we see how it has added an extra 'chunk' into the existing basic-shared-policy
```
admin@ncs(config-shared-route-policy-alwaysoniosxr/11:11)# shared-route-policy alwaysoniosxr 33:33 set 44:44
admin@ncs(config-shared-route-policy-alwaysoniosxr/33:33)# commit dry-run
cli {
    local-node {
        data +basic-shared-policy alwaysoniosxr VRF-shared-1-EXP-RPL {
             +    chunk [ "  if community matches-any (11:11) then\r\n    set extcommunity rt (22:22) additive\r\n  endif\r\n" "  if community matches-any (33:33) then\r\n    set extcommunity rt (44:44) additive\r\n  endif\r\n" ];
             +}
             +shared-route-policy alwaysoniosxr 11:11 {
             +    set 22:22;
             +}
             +shared-route-policy alwaysoniosxr 33:33 {
             +    set 44:44;
             +}
              devices {
                  device alwaysoniosxr {
                      config {
             +            route-policy VRF-shared-1-EXP-RPL {
             +                value "  if community matches-any (11:11) then\r\n    set extcommunity rt (22:22) additive\r\n  endif\r\n  if community matches-any (33:33) then\r\n    set extcommunity rt (44:44) additive\r\n  endif\r\n";
             +            }
                      }
                  }
              }
    }
}
admin@ncs(config-shared-route-policy-alwaysoniosxr/33:33)# commit dry-run outformat native
native {
    device {
        name alwaysoniosxr
        data route-policy VRF-shared-1-EXP-RPL
               if community matches-any (11:11) then
                 set extcommunity rt (22:22) additive
               endif
               if community matches-any (33:33) then
                 set extcommunity rt (44:44) additive
               endif
              end-policy
             !
    }
}
admin@ncs(config-shared-route-policy-alwaysoniosxr/33:33)# commit
Commit complete.
```

Also when we delete a service the policy is adjusted appropriately
```
admin@ncs(config-shared-route-policy-alwaysoniosxr/33:33)# top
admin@ncs(config)# no shared-route-policy alwaysoniosxr 11:11
admin@ncs(config)# commit dry-run outformat native
native {
    device {
        name alwaysoniosxr
        data route-policy VRF-shared-1-EXP-RPL
               if community matches-any (33:33) then
                 set extcommunity rt (44:44) additive
               endif
              end-policy
             !
    }
}
admin@ncs(config)# commit
Commit complete.
```
