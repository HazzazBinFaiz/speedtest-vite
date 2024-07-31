<script>

    import prettyBytes from "pretty-bytes";

    let speed = 'Ready';
    let processing = false;

    let servers = [
        {
            name: 'JSON Placeholder',
            url: 'https://jsonplaceholder.typicode.com/todos',
            speed: 0,
            speeds: [],
            speedText: '...',
            reachable: true,
        },
        {
	    name: '1 MB Payload',
            url: './1mb',
	    speed: 0,
	    speedText: '...',
	    reachable: true
	},
    ];

    const totalTrip = 5;

    const testSpeed = async () => {
      processing = true;
      speed = '...';

      debugger
      for (let i = 0; i < servers.length; i++) {
          servers[i].speeds = [];
          for (let trip = 0; trip < totalTrip; trip++) {
              speed = `..${servers.length-i}.${totalTrip-trip}..`
              const start = new Date().getTime();
              try {
                  let r = await fetch(servers[i].url)
                  const length = (await r.text()).length;
                  let seconds = ((new Date().getTime()) - start) / 1000;
                  servers[i].speeds.push(Math.floor(length / seconds));
              } catch (e) {
                  servers[i].reachable = false;
                  servers[i].speedText = 'Unreachable';
                  break;
              }
          }
          if (servers[i].reachable) {
              servers[i].speed = servers[i].speeds.reduce((a, b) => a+b, 0) / servers[i].speeds.length;
              servers[i].speedText = prettyBytes(servers[i].speed);
          }
      }

      const filtered = servers.filter(s => s.reachable);
      if (filtered.length > 0) {
          speed = prettyBytes(filtered.map(s => s.speed).reduce((a,b)=>a+b, 0) / servers.length);
          servers = [...servers]
      } else {
          speed = '😭'
      }
      processing = false;
    }

</script>

<main>
    <div class="flex flex-col items-center justify-evenly h-screen bg-background p-8">
        <div class="text-center py-16">
            <div class="text-8xl font-bold text-slate { processing ? 'animate animate-pulse' : '' }">{ speed }</div>
            <button on:click={testSpeed}
                    class="mt-16 inline-flex items-center justify-center whitespace-nowrap rounded-md ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-slate-800 text-slate-700 hover:bg-slate-900 px-8 py-4 text-lg text-white font-medium">
                Test Speed
            </button>
        </div>
        <div class="mt-12 w-full max-w-md">
            <h3 class="text-lg font-medium mb-4">Server Speeds</h3>
            <div class="max-h-[300px] overflow-auto">
                <div class="relative w-full overflow-auto">
                    <table class="w-full caption-bottom text-sm">
                        <thead class="border-b">
                        <tr class="border-b transition-colors">
                            <th class="h-12 px-4 text-left align-middle font-medium text-muted-foreground">
                                Server
                            </th>
                            <th class="h-12 px-4 align-middle font-medium text-muted-foreground text-right">
                                Speed
                            </th>
                        </tr>
                        </thead>
                        <tbody>
                        {#each servers as server }
                            <tr class="border-b transition-colors { server.reachable ? '' : 'text-red-500' }">
                                <td class="p-4 align-middle">{server.name}</td>
                                <td class="p-4 align-middle text-right">{server.speedText}</td>
                            </tr>
                        {/each}
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</main>
