import java.util.*;

// 1. PROCESS DATA & SHARED HELPERS
class Process {
    int id, at, bt, prio, rem, ct, tat, wt;

    Process(int id, int at, int bt, int prio) {
        this.id = id; this.at = at; this.bt = bt; this.prio = prio; this.rem = bt;
    }

    // Logs 1 tick of CPU time; returns 1 if the process finishes, 0 otherwise
    static int tick(Process c, int t, List<String> log) {
        log.add(c == null ? "Idle" : "P" + c.id);
        if (c == null || --c.rem > 0) return 0;
        c.ct = t + 1; c.tat = c.ct - c.at; c.wt = c.tat - c.bt;
        return 1;
    }

    // Prints Gantt chart, table, averages, CPU utilization, and resets rem for next run
    static void print(List<Process> list, String title, List<String> log) {
        String gantt = "";
        for (int t = 0; t < log.size(); t++)
            if (t == 0 || !log.get(t).equals(log.get(t - 1))) gantt += "(" + t + ") | " + log.get(t) + " | ";

        int time = log.size(), n = list.size();
        double wt = 0, tat = 0, bt = 0;

        System.out.println("\n=== " + title + " ===\nGantt Chart: " + gantt + "(" + time + ")\nID\tAT\tBT\tPrio\tCT\tTAT\tWT");
        for (Process p : list) {
            bt += p.bt; tat += p.tat; wt += p.wt; p.rem = p.bt; // Auto-reset rem
            System.out.printf("P%d\t%d\t%d\t%d\t%d\t%d\t%d\n", p.id, p.at, p.bt, p.prio, p.ct, p.tat, p.wt);
        }
        System.out.printf("Avg WT : %.0f / %d = %.2f\nAvg TAT: %.0f / %d = %.2f\nCPU Utilization: (%.0f / %d) * 100 = %.2f%%\n",
            wt, n, wt / n, tat, n, tat / n, bt, time, (bt / time) * 100);
    }
}

// 2. SRTF CLASS
class SRTF {
    void solve(List<Process> list) {
        List<String> log = new ArrayList<>();
        for (int t = 0, done = 0; done < list.size(); t++) {
            int now = t;
            Process cur = list.stream().filter(p -> p.at <= now && p.rem > 0)
                .min((a, b) -> a.rem != b.rem ? a.rem - b.rem : a.at - b.at).orElse(null);
            done += Process.tick(cur, t, log);
        }
        Process.print(list, "SRTF", log);
    }
}

// 3. PREEMPTIVE PRIORITY CLASS
class PriorityPreemptive {
    void solve(List<Process> list) {
        List<String> log = new ArrayList<>();
        for (int t = 0, done = 0; done < list.size(); t++) {
            int now = t;
            Process cur = list.stream().filter(p -> p.at <= now && p.rem > 0)
                .min((a, b) -> a.prio != b.prio ? a.prio - b.prio : a.at - b.at).orElse(null);
            done += Process.tick(cur, t, log);
        }
        Process.print(list, "Preemptive Priority", log);
    }
}

// 4. ROUND ROBIN CLASS
class RoundRobin {
    void solve(List<Process> list, int q) {
        List<String> log = new ArrayList<>();
        Queue<Process> rq = new LinkedList<>();
        Process cur = null;
        for (int t = 0, done = 0, qt = 0; done < list.size(); t++, qt++) {
            for (Process p : list) if (p.at == t) rq.add(p);
            if (cur != null && qt == q) { rq.add(cur); cur = null; }
            if (cur == null && !rq.isEmpty()) { cur = rq.poll(); qt = 0; }
            if (Process.tick(cur, t, log) == 1) { done++; cur = null; }
        }
        Process.print(list, "Round Robin (Q=" + q + ")", log);
    }
}

// 5. MAIN CLASS
public class Main {
    public static void main(String[] args) {
        List<Process> list = Arrays.asList(
            new Process(1, 0, 8, 2),
            new Process(2, 1, 4, 1),
            new Process(3, 2, 9, 3),
            new Process(4, 3, 5, 2)
        );
        new SRTF().solve(list);
        new PriorityPreemptive().solve(list);
        new RoundRobin().solve(list, 3);
    }
}
