<style>
  ul, li {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }
  
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  
  .noti-register-box {
    display: inline-block;
  }
  .noti-register-box .clock {
    display: inline-block;
  }

  .noti-register-box .clock .number {
    font-size: 40px;
  }

  .noti-register-box .clock .number input {
    width: 85px;
  }
  .noti-register-box .noti-msg {
    display: flex;
    flex-grow: 1;
    width: 100%;
    font-size: 20px;
  }

  .noti-register-box .register-btn {
    color: #fff;
    background: #ff3e00;
    width: 100%;
    line-height: 30px;
  }
  
</style>

<script>
  import { onMount } from 'svelte';

  let hours = '00';
  let minutes = '00';
  let seconds = '00';
  
  let msg = '';
  let notiPermission = false;

  let alarmList = [];
  
  function handleRegister() {
    const newAlarm = {
      time: `${hours}:${minutes}:${seconds}`,
      msg: msg,
      done : false,

    };

    const check = alarmList.length > 0 ? alarmList.filter((v) => v.time === newAlarm.time).length : 0;

    if (check === 0) {
      alarmList = [newAlarm, ...alarmList]
    } else {
      console.log('이미 존재합니다');
    }
  }

  function handleDeleteTime(alarm) {
    alarmList = alarmList.filter((a) => a.time !== alarm.time);
  }

  onMount(() => {
    console.log(  notiPermission);
    if (Notification &&   notiPermission !== 'granted') {
      Notification.requestPermission(function (status) {
          notiPermission = status;
      });
    }

    const interval = 1000; // ms
    let expected = Date.now() + (interval - (Date.now() % interval));

    setTimeout(step, interval - (Date.now() % interval));
    function step() {
      const now = new Date();
      const hour = (now.getHours() < 10 ? '0'+now.getHours() : now.getHours());
      const minutes = (now.getMinutes() < 10 ? '0'+now.getMinutes() : now.getMinutes());
      const seconds = (now.getSeconds() < 10 ? '0'+now.getSeconds() : now.getSeconds());
      const strTime = hour + ':'+ minutes + ':'+ seconds;
      //console.log(strTime);

      var dt = now.getTime() - expected; // the drift (positive for overshooting)
      // 기대한 시간이 타이머 간격보다 클때 예외처리
      if (dt > interval) {
        console.log('------');
      }

      alarmList.map((alarm) => {
        //console.log(alarm.time);
        
        if (!alarm.done && alarm.time <= strTime) {
          //console.log(`[${alarm.time}] ${alarm.msg}`);
          var notification = new Notification('Noti Time', { body: `[${alarm.time}] ${alarm.msg}` });

          notification.onclose = function() {
            console.log(strTime + " >> onclose");
            alarm.done = true;
          };
        }
      });

      expected += interval;
      setTimeout(step, Math.max(0, interval - dt)); // take into account drift
    }

    return () => {
      clearInterval(interval);
    }
  })
</script>
<main>
  <h1>Noti Time</h1>

  {#if notiPermission !== 'granted'}
    <p>알림이 꺼져 있습니다. </p>
  {/if}
  <div class='noti-register-box'>
    <div>
      <div class='clock'>
        <span class='number'>
          <input bind:value={hours}/>
        </span>
        &nbsp; : &nbsp; 
        <span class='number'>
          <input bind:value={minutes}/>
        </span>
        &nbsp; : &nbsp; 
        <span class='number'>
          <input bind:value={seconds}/> 
        </span>
      </div>
      <input class='noti-msg' bind:value={msg}/>
    </div>
    <button class='register-btn' on:click={handleRegister}>등록</button>
  </div>

  <ul>
    {#each alarmList as alarm}
      <li>{alarm.time}  {alarm.msg} <button on:click={() => handleDeleteTime(alarm)}>삭제</button></li>
    {:else}
      <li>알람을 등록해 주세요.</li>
    {/each}
  </ul>
  
</main>
