PUBLIC int do_lottery()
{
	struct schedproc *rmp;
 	int proc_nr;
 	int rv;
 	int lucky;
 	int old_priority;
 	int flag = -1;
 	int nTickets = 0;
 
 	for (proc_nr=0, rmp=schedproc; proc_nr < NR_PROCS; proc_nr++, rmp++) {
 		if ((rmp->flags & IN_USE) && PROCESS_IN_USER_Q(rmp)) {
 			if (USER_Q == rmp->priority) {
 				nTickets += rmp->ticketsNum;
 			}
 		}
 	}
 
 	lucky = nTickets ? rand() % nTickets : 0;
 	for (proc_nr=0, rmp=schedproc; proc_nr < NR_PROCS; proc_nr++, rmp++) {
 		if ((rmp->flags & IN_USE) && PROCESS_IN_USER_Q(rmp) &&
 				USER_Q == rmp->priority) {
 			old_priority = rmp->priority;
 			 rmp->priority = USER_Q; 
 			if (lucky >= 0) {
 				lucky -= rmp->ticketsNum;
 				
 				   printf("lucky - %d = %d\n", rmp->ticketsNum, lucky);
 				 
 				if (lucky < 0) {
 					rmp->priority = MAX_USER_Q;
 					flag = OK;
 					printf("endpoint %d\n", rmp->endpoint); 
 				}
 			}
 			if (old_priority != rmp->priority) {
 				schedule_process(rmp);
 			}
 		}
 	}
 	
 	for (proc_nr=0, rmp=schedproc; proc_nr < NR_PROCS; proc_nr++, rmp++) {
 		if ((rmp->flags & IN_USE) && PROCESS_IN_USER_Q(rmp)) {
 			if (USER_Q == rmp->priority)
 				count_17++;
 			else if (MAX_USER_Q == rmp->priority)
 				count_16++;
 		}
 	}
 	printf("in 16: %d; in 17: %d\n", count_16, count_17); 
 	
 	 printf("do_lottery OK? %d lucky=%d\n", flag, lucky); 
 	return nTickets ? flag : OK;
 }
 

 PUBLIC int set_priority(int ntickets, struct schedproc* p)
 {
 	int add;
  	add = p->ticketsNum + ntickets > 100 ? 100 - p->ticketsNum : ntickets;
 	add = p->ticketsNum + ntickets < 1 ? 1 - p->ticketsNum: add;
 	p->ticketsNum += add;
 	return add;
