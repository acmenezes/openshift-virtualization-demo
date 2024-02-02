
`mstconfig -d 0b:00.0 query`

```                                                                                                                                                   
Device #1:                                                                                                                                         
----------                                                                                                                                         
                                                                                                                                                   
Device type:    ConnectX4LX                                                                                                                        
Name:           MCX4121A-ACA_Ax                                                                                                                    
Description:    ConnectX-4 Lx EN network interface card; 25GbE dual-port SFP28; PCIe3.0 x8; ROHS R6                                                
Device:         0b:00.0                                                                                                                            
                                                                                                                                                   
Configurations:                              Next Boot                                                                                             
         MEMIC_BAR_SIZE                      0                                                                                                     
         MEMIC_SIZE_LIMIT                    _256KB(1)                                                                                             
         FLEX_PARSER_PROFILE_ENABLE          0                                                                                                     
         FLEX_IPV4_OVER_VXLAN_PORT           0                                                                                                     
         ROCE_NEXT_PROTOCOL                  254                                                                                                   
         NON_PREFETCHABLE_PF_BAR             False(0)                                                                                              
         VF_VPD_ENABLE                       False(0)                                                                                              
         STRICT_VF_MSIX_NUM                  False(0)                                                                                              
         VF_NODNIC_ENABLE                    False(0)                                                                                              
         NUM_OF_VFS                          0               <----- zero VFs by default
         SRIOV_EN                            False(0)        <----- SRIOV disabled by default
         PF_LOG_BAR_SIZE                     5                                                                                                     
         VF_LOG_BAR_SIZE                     0                                                                                                     
         NUM_PF_MSIX                         63                                                                                                    
         NUM_VF_MSIX                         11                                                                                                    
         INT_LOG_MAX_PAYLOAD_SIZE            AUTOMATIC(0)                                                                                          
         PARTIAL_RESET_EN                    False(0)                                                                                              
         SW_RECOVERY_ON_ERRORS               False(0)                                                                                              
         RESET_WITH_HOST_ON_ERRORS           False(0)                                                                                              
         CQE_COMPRESSION                     BALANCED(0)                                                                                           
         IP_OVER_VXLAN_EN                    False(0)                                                                                              
         UCTX_EN                             True(1)                                                                                               
         PCI_ATOMIC_MODE                     PCI_ATOMIC_DISABLED_EXT_ATOMIC_ENABLED(0)
         LRO_LOG_TIMEOUT0                    6                                                                                                     
         LRO_LOG_TIMEOUT1                    7                                                                                                     
         LRO_LOG_TIMEOUT2                    8                                                                                                     
         LRO_LOG_TIMEOUT3                    13

< ..... TRUNCATED ..... >
```

`lspci | grep -i mellanox`

```
0b:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
0b:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
```




