# PG-Streaming-replication

Setup streaming replication with slot.  
1 Change parameters in primary server.  
alter system set wal_level to 'replica';  
alter system set listen_addresses to '*';  
alter system set 'synchronous_commit' to 'off';  
alter system set full_pate_writes to 'on';  
alter system set wal_compression to 'on';  
select pg_reload_conf();  
show wal_level;  
show listen_addresses;  
show synchronous_commit;  
show full_pate_writes;  
show wal_compression;  

2
