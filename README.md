# shared-route-policy
An NSO service package to manage a route-policy that is being used by many service instances

Tested on IOS-XR

For a service like this

admin@ncs(config)# show full-configuration shared-route-policy
shared-route-policy alwaysoniosxr
 match-set 123:456
  set 333:444
 !
 match-set 2:2
  set 3:3
 !
!
This produces a route policy like this

RP/0/RP0/CPU0:XR_SANDBOX#show running-config route-policy VRF-shared-1-EXP-RPL
Wed Oct  9 14:15:34.202 UTC
route-policy VRF-shared-1-EXP-RPL
  if community matches-any (123:456) then
    set extcommunity rt (333:444) additive
  endif
  if community matches-any (2:2) then
    set extcommunity rt (3:3) additive
  endif
end-policy
!

So if we look at this service in xml form

admin@ncs(config)# show full-configuration shared-route-policy | display xml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <shared-route-policy xmlns="http://example.com/shared-route-policy">
    <device>alwaysoniosxr</device>
    <match-set>
      <match>123:456</match>
      <set>333:444</set>
    </match-set>
    <match-set>
      <match>2:2</match>
      <set>3:3</set>
    </match-set>
  </shared-route-policy>
</config>

Any service can create its own match-set within its templates and as long as the match keys differ, then the services will not overwrite each other.
You can of course use <match-set tages='create'> to ensure that you are creating a new match and not overwriting an existing one.
