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

2 Create replication user in primary server.  
create role streaming_rep_slot with replication password '${password}' login;  

3 Test connection from standby server. Add the following info to pg_hba.conf in primary server if failed.  
host all all ${ip of standby}/32 md5  

4 Backup the primary server.  
pg_basebackup -D ${backup_dir} -vP -W -X -C --slot='streaming_replication_slot'  
#Backup and create a slot named streaming_replication_slot.  

5 Recover in the standby server.  

6 Setup the streaming replication with slot in the standby server.  
touch ${PG_dir}/data/standby.signal  
cat <<+ >> ${PG_dir}/data/postgresql.conf  
primary_conninfo = 'host=${IP of primary} port=5432 user=streaming_rep password=${password}'  
primary_slot_name = 'streaming_replication_slot'  
+

7 Boot up the standby server.  The replication should work well.

8 Post-checking in the primary server.  
select * from pg_replciation_slots where slot_name='streaming_repliation_slots';
select * from pg_stat_replication;
select * from pg_stat_activity where usename='streaming_rep';

9 Check the replication lag in the primary and standby server.  
In primary server.  
select redo_lsn, slot_name, restart_lsn, round((redo_lsn-restart_lsn)/(1024*1024*1024), 2) as GB_behild  
from pg_control_checkpoint(), pg_replication_slots  
where slot_name='streaming_repliation_slots'  

In standby server.  
select *, pg_last_xact_replay_timestamp(), now(), round(extract(epoch from (age(now), pg_last_eact_replay_timestamp()))) as replication_lag_seconds, pg_last_wal_replay_lsn()  
from pg_stat_wal_receiver;  

10 Reduce the replication lag.  
10.1 Reduce the transaction in the primary server.  
10.2 Increase the network bandwidth between the primary and standby server.  Usually, network is the bottleneck.  
10.3 Compress WAL in the primary server.  
alter system set full_pate_writes to 'on';  
alter system set wal_compression to 'on';  
10.4 Change the priority level of walsender and walreceiver from OS. 
Check the priority level first, 'ps -o ni ${PID}'  
Change the priority, 'renice -n -10 -p ${PID}'  

11 Continue monitoring the replication lag, the size of WAL, the process efficiency of walsender and walreceiver.  




