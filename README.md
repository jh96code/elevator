//알고리즘 과제(엘리베이터 시뮬레이션)_20260604

import java.util.ArrayList;
import java.util.List;
import static java.lang.Math.*;

// 1. Elevator 클래스 (엘리베이터 변수)
class Elevator {
    public String name; // 엘리베이터명 (A:1~33층 / B:34~66층 / C:67~100층)
    public int minFloor; // 최저층수
    public int maxFloor; // 최고층수
    public int chu = 300; // 균형추 무게
    public int curFloor = 0; // 현재층수
    public String status = ""; // 엘리베이터 상태 (UP / DOWN / STOP / NO_USE)
    public List<Integer> visitList = new ArrayList<>(); // 머무는 층수 리스트
    public final int maxWeight = 900; // 최대 적재량
    public int curWeight = 0; // 현재 적재량
    public int cost = 0; // 엘리베이터 배정 비용
    
    public float evEnergy = 0.0f; // 누적 에너지 소모량
    public float upEnergy = 1.5f; // 상승시 에너지 소모량
    public float upWeightEnergy = 1.0f; //상승시 적재량 초과시 소모량
    public float downEnergy = 1.0f; // 하강시 에너지 소모량
    public float downWeightEnergy = 0.5f; //하강시 적재량 초과시 소모량
    public float startEnergy = 3.0f; // 하강 에너지 소모량
    public int evTime = 0; // 누적 소요시간
    public int visitTime = 10; // 층수에 머무는 시간
    public int startTime = 3; // 출발할때 걸리는 시간
    public int upTime = 2; // 상승 소요시간
    public int downTime = 1; // 하강 소요시간
    

    //생성자
    public Elevator(String name, int minFloor, int maxFloor) {
        this.name = name;
        this.minFloor = minFloor;
        this.maxFloor = maxFloor;
    }
}


// 2. ElevatorController 클래스 (연산 담당)
class ElevatorController {

    //엘리베이터 객체 생성
    public List<Elevator> elevators;
    public ElevatorController() {
        this.elevators = new ArrayList<>();
        this.elevators.add(new Elevator("A", 1, 33));
        this.elevators.add(new Elevator("B", 34, 66));
        this.elevators.add(new Elevator("C", 67, 100));
    }

    // 엘리베이터 이동시 에너지 소모량 계산 함수
    public float calculateEnergy(Elevator ev, boolean isStarting) {
        float baseEnergy = 0.0f;
        float extraEnergy = 0.0f;

        if (ev.curWeight > ev.chu) {
            if (ev.status.equals("UP")) {
                baseEnergy = ev.upEnergy;
                extraEnergy = (float) (((ev.curWeight - ev.chu) / 100.0) * ev.upWeightEnergy);
            } else if (ev.status.equals("DOWN")) {
                baseEnergy = ev.downEnergy;
                extraEnergy = (float) (((ev.curWeight - ev.chu) / 100.0) * ev.downWeightEnergy);
            }
        }

        float totalStepEnergy = baseEnergy + extraEnergy;

        if (isStarting) {
            totalStepEnergy += ev.startEnergy;
        }

        return totalStepEnergy;
    }

    //엘리베이터 소요시간 함수 (상승:2초 / 하강:1초)
    public int calculateTime(Elevator ev, boolean isStarting) {
        int totalTime = 0;

        if (ev.status.equals("UP")) {
            totalTime = ev.upTime;
        } else if (ev.status.equals("DOWN")) {
            totalTime = ev.downTime;
        }

        if (isStarting) {
            totalTime += ev.startTime; 
        }

        return totalTime;
    }

    // 엘리베이터별 배정 비용 계산 함수 (대기시간으로 계산)
    public int calculateCost(Elevator ev, int callFloor, int visitTime) {
        int tempFloor = ev.curFloor;
        int totalTime = 0;

        // 1. 현재 작동중인 엘리베이터의 종료까지의 시간 계산
        for (int i = 0; i < ev.visitList.size(); i++) {
            int nextFloor = ev.visitList.get(i);
            int distance = Math.abs(tempFloor - nextFloor);

            if (nextFloor > tempFloor) {
                totalTime += (distance * ev.upTime) + ev.startTime;
            } else if (nextFloor < tempFloor) {
                totalTime += (distance * ev.downTime) + ev.startTime;
            }
            totalTime += visitTime;
            tempFloor = nextFloor;
        }

        // 2. 최종 점수 계산
        int distance = Math.abs(tempFloor - callFloor);
        if (callFloor > tempFloor) {
            totalTime += (distance * ev.upTime) + ev.startTime;
        } else if (callFloor < tempFloor) {
            totalTime += (distance * ev.downTime) + ev.startTime;
        }

        if (callFloor < ev.minFloor || callFloor > ev.maxFloor) {
            totalTime += 30;
        }
        String role = (ev.minFloor <= callFloor && callFloor <= ev.maxFloor) ? "관할구역" : "비관할구역";

        System.out.println(ev.name + "호기: " + totalTime + "점(" + role + ")");
        
        return totalTime;
    }

    // 호출시 최적의 엘리베이터 배정 함수 (대기시간 최소 엘리베이터 배정)
    public Elevator registerCall(int callFloor) {
        System.out.println("[" + callFloor + "층에서 호출 발생]\n");

        // 1. 각 엘리베이터의 배정 비용 계산
        System.out.println("[엘리베이터 배정 비용 계산 결과]");
        for (Elevator ev : this.elevators) {
            ev.cost = calculateCost(ev, callFloor, ev.visitTime);
        }

        // 2. 비용이 가장 적은 엘리베이터 선택
        Elevator bestEv = null;
        int minCost = Integer.MAX_VALUE;
        for (Elevator ev : this.elevators) {
            if (ev.cost < minCost) {
                minCost = ev.cost;
                bestEv = ev;
            }
        }

        // 3. 호출 층수를 목적지 리스트에 저장
        if (bestEv != null) {
            System.out.println("[" + bestEv.name + "호기 최종 배정] \n");
            bestEv.visitList.add(callFloor); 
        }

        return bestEv;
    }

    //엘리베이터 운행 시뮬레이션 함수
    public void evSimulation(Elevator targetEv) {
        int beforeFloor = targetEv.curFloor;
        boolean isStarting = true;

        System.out.println("[엘리베이터 시뮬레이션]");
        while (!targetEv.visitList.isEmpty()) {
            int targetFloor = targetEv.visitList.get(0);

            // 방향 판단 및 도달 체크
            if (targetFloor > targetEv.curFloor) {
                targetEv.status = "UP";
            } else if (targetFloor < targetEv.curFloor) {
                targetEv.status = "DOWN";
            } else {
                targetEv.visitList.remove(0);
                if (!targetEv.visitList.isEmpty()) {
                    targetEv.status = "STOP";
                    targetEv.evTime += targetEv.visitTime;
                    
                    isStarting = true;
                    System.out.println(targetEv.curFloor + "층 " + targetEv.status + " (소요 시간: " + targetEv.visitTime + "초)");
                    continue;
                } else {
                    // 더 이상 갈 곳이 없다면 최종 종료
                    targetEv.status = "NO_USE";
                    System.out.println("[호출 층수 도착] (누적 소요 에너지: " + targetEv.evEnergy + " / 누적 소요 시간: " + targetEv.evTime + "초)\n");         
                    break;
                }
            }

            float stepEnergy = calculateEnergy(targetEv, isStarting);
            int stepTime = calculateTime(targetEv, isStarting);

            targetEv.evEnergy += stepEnergy;
            targetEv.evTime += stepTime;
            
            beforeFloor = targetEv.curFloor; // [보완] 다음 루프 출력을 위해 현재 층수 백업
            targetEv.curFloor += (targetEv.status.equals("UP")) ? 1 : -1;

            if (isStarting) {
                isStarting = false;
            }

            System.out.println(beforeFloor + "층 => " + targetEv.curFloor + "층 " + targetEv.status + " (소요 에너지: " + stepEnergy + " / 소요 시간: " + stepTime + "초)");
        }
    }
}

// 3. Main 클래스 (실행부 및 실시간 센서 시뮬레이션)
public class Main {
    public static void main(String[] args) {
        ElevatorController controller = new ElevatorController();

        controller.elevators.get(0).curFloor = 1;   
        controller.elevators.get(1).curFloor = 34;  
        controller.elevators.get(2).curFloor = 70;
        controller.elevators.get(2).status = "UP";
        controller.elevators.get(2).curWeight = 400;
        controller.elevators.get(2).visitList.add(90);
        int callFloor = 85;
        Elevator targetEv = controller.registerCall(callFloor);

        controller.evSimulation(targetEv);
    }
}
