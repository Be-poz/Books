서브클래싱과 서브타이핑
-
본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.  

커맨드 패턴을 사용하면 어떤 작업을 요청한 쪽하고 그 작업을 처리한 쪽을 분리시킬 수가 있다.  

만능 리모컨을 만든다고 가정하자. 이 리모컨에서는 여러 전자기기의 컨트롤이 가능하다고 한다면, 클래스마다 메서드도 제각기일 것이고 추가/수정에 있어
굉장히 많은 비용이 들어갈 것이다.  

커맨드 객체는 이를 해결할 수 있다.  


```java
public interface Command{
    public void execute();
}

public class LightOnCommand implements Command{
    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }
    public void execute(){
        light.on();
    }
}
```
Command 인터페이스는 execute() 메서드 단 하나만 가진다.  
전등을 켜기 위한 커맨드 클래스 LightOnCommand를 구현하였다.  
생성자에 이 커맨드 객체로 제어할 특정 전등에 대한 정보가 전달되고, 이 객체는 light 라는 인스턴스 변수에 저장이 된다.
execute()메서드가 호출되면 light 객체가 바로 그 요청에 대한 리시버(Receiver)가 된다.  

```java
public class SimpleRemoteControl{
    Command slot;
    
    public SimpleRemoteControl(){}
    
    public void setCommand(Command command){
        slot = command;
    }
    
    public void buttonWasPressed(){
        slot.execute();
    }
}

public class RemoteControlTest{
    public static void main(String[] args) {
        SimpleRemoteControl remote = new SimpleRemoteControl();
        Light light = new Light();
        LightOnCommand lightOn = new LightOnCommand(light);
        
        remote.setCommand(lightOn);
        remote.buttonWasPressed();
    }
}
```
커맨드를 집어넣을 슬롯이 하나 밖에 없는 리모콘이라고 가정하고 SimpleRemoteControl 클래스를 구현하였다. Command를 set하고 버튼이 눌리면 execute()
메서드를 실행한다. RemoteControlTest에서는 요청을 받아서 처리할 리시버(Receiver)인 Light 객체를 생성하여 커맨드 객체를 만들고,  
SimpleRemoteControl에 넣어준다. 여기서 이 리모트컨트롤이 인보커 역할을 한다. 리모컨의 버튼이 눌리면 execute()메서드가 실행될 것이다.  

>커맨드패턴  
>커맨드 패턴을 이용하면 요구 사항을 객체로 캡슐화 할 수 있으며, 매개변수를 써서 여러 가지 다른 요구 사항을 집어넣을 수도 있다.  
>또한 요청 내역을 큐에 저장하거나 로그로 기록할 수도 있으며, 작업취소 기능도 지원 가능하다.

![command1](https://user-images.githubusercontent.com/45073750/94342861-17676300-004f-11eb-9e09-a3872b46dda3.PNG)
* 클라이언트는 ConcreteCommand를 생성하고 Receiver를 설정한다.  
* 리시버는 요구 사항을 수행하기 위해 어떤 일을 처리해야 하는지 알고 있는 객체이다.
* ConcreteCommand는 특정 행동과 리비서 사이를 연결해준다. 인보커에서 execute() 호출을 통해 요청을 하면 ConcreteCommand 객체에서 리시버에 있는 메서드를 호출함으로써 그 작업을 처리한다.  
* Command는 모든 커맨드 객체에서 구현해야 하는 인터페이스이다.
* 인보커에는 명령이 들어있으며, execute()메서드를 호출함으로써 커맨드 객체에게 특정 작업을 수행해 달라는 요구를 하게된다.  

```java
public class RemoteControl{
    Command[] onCommands;
    Command[] offCommands;
    
    public RemoteControl(){
        onCommands = new Command[7];
        offCommands = new Command[7];
        
        Command noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
    }
}
```
이번에는 7개의 슬롯을 만들어서 슬롯에 Command 를 넣고, 슬롯 값에 따른 onButtonPushed 와 offButtonPushed를 구현해보았다.  
on/off 로 나뉘었다고 크게 다른게 아니라 LightOffCommand, LightOnCommand 이렇게 나누어서 만들어주면 된다.  
```java
public class StereoOnWithCDCommand implements Command {
    Stereo stereo;

    public StereoOnWithCDCommand(Stereo stereo) {
        this.stereo = stereo;
    }
    public void execute(){
        stereo.on();
        stereo.setCD();
        stereo.setVolume(11);
    }
}
```
다음과 같이 오디오를 켜고 끌 때 CD재생까지 되는 약간의 응용도 할 수 있다.  
```java
public class NoCommand implements Command {
    public void execute(){}
}
```
NoCommand 클래스는 다음과 같이 execute()메서드가 비어있는 클래스이다. onButtonWasPushed() 메서드를 구현할 때에 onCommands[slot]의 null 여부를
체크하기에는 귀찮아지므로 NoCommand 클래스를 따로 구현해준 것이다. 일종의 널 객체이다.  

undo() 메서드를 추가해주고 싶다면 Command interface에 public void undo()를 추가해주고 이를 구현하는 ConcreteCommand 클래스에서 상세구현을 해주면 될 것이다.  
그리고 RemoteControl 클래스에 마지막에 누른 버튼을 기록해주어야 제대로 undo 기능을 이용할 수 있을 것이다.  
```java
public interface Command{
    public void execute();
    public void undo();
}

public class LightOnCommand implements Command{
    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }
    public void execute(){
        light.on();
    }
    public void undo(){
        light.off();
    }
}

public class RemoteControlWithUndo{
    Command[] onCommands;
    Command[] offCommands;
    Command undoCommand;
    
    public RemoteControlWithUndo(){
        onCommands = new Command[7];
        offCommands = new Command[7];
        
        Command noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }
    
    public void undoButtonWasPushed(){
        undoCommand.undo();
    }
}
```
다음과 같이 말이다.  
버튼 한 개로 여러 기능을 조종하게끔 할 수도 있다.  
execute() 메서드내에 commands를 돌면서 execute()를 실행해주면 될 것이다.  
****
