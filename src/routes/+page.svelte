<script lang="ts">                                                             
  import { onMount } from 'svelte';                                            
  import { Socket } from 'phoenix';                                            
                                                                               
  let socketStatus: string = 'Connecting...';                                  
  let channelStatus: string = 'Joining channel...';                            
  let channelResponse: any = null;                                             
  let channel: any = null; // To store the channel object                      
                                                                               
  onMount(() => {                                                              
    // Ensure this path matches your Phoenix endpoint's socket path            
    // And the host/port matches your Phoenix server.                          
    // For development, Phoenix usually runs on http://localhost:4000          
    const phoenixSocketPath = 'ws://localhost:4000/socket';                    
                                                                               
    // You might pass params like a user token if your UserSocket requires it  
    // const socket = new Socket(phoenixSocketPath, { params: { user_token:    
// "some_token" } });                                                             
    const socket = new Socket(phoenixSocketPath, {});                          
                                                                               
    socket.onOpen(() => {                                                      
      socketStatus = 'Socket Connected!';                                      
      console.log('Phoenix Socket: Opened');                                   
                                                                               
      // Join the channel once the socket is open                              
      // The topic "stock:lobby" must match what you defined in StockChannel.join/3
      channel = socket.channel('stock:lobby', { client: 'SvelteApp' });        
                                                                               
      channel.join()                                                           
        .receive('ok', (resp: any) => {                                        
          channelStatus = 'Channel Joined Successfully!';                      
          channelResponse = resp;                                              
          console.log('Phoenix Channel: Joined stock:lobby', resp);            
                                                                               
          // Optional: Send a test "ping" message                              
          channel.push('ping', { message: 'Hello from Svelte!' })              
            .receive('ok', (pongResp: any) => {                                
              console.log('Phoenix Channel: Received pong', pongResp);         
            })                                                                 
            .receive('error', (reasons: any) => {                              
              console.error('Phoenix Channel: Ping failed', reasons);          
            })                                                                 
            .receive('timeout', () => {                                        
              console.warn('Phoenix Channel: Ping timed out');                 
            });                                                                
        })                                                                     
        .receive('error', (resp: any) => {                                     
          channelStatus = 'Failed to join channel.';                           
          channelResponse = resp;                                              
          console.error('Phoenix Channel: Unable to join stock:lobby', resp);  
        })                                                                     
        .receive('timeout', () => {                                            
          channelStatus = 'Channel join timed out.';                           
          console.warn('Phoenix Channel: Join timed out for stock:lobby');     
        });                                                                    
                                                                               
      channel.on("some_event_from_server", (payload: any) => {                 
        console.log("Received event from server:", payload);                   
        // Handle the broadcasted event here                                   
      });                                                                      
                                                                               
    });                                                                        
                                                                               
    socket.onError((error: any) => {                                           
      socketStatus = `Socket Error: ${error}`;                                 
      console.error('Phoenix Socket: Error', error);                           
    });                                                                        
                                                                               
    socket.onClose(() => {                                                     
      socketStatus = 'Socket Closed.';                                         
      console.log('Phoenix Socket: Closed');                                   
    });                                                                        
                                                                               
    // Attempt to connect the socket                                           
    socket.connect();                                                          
                                                                               
    // Cleanup when the component is destroyed                                 
    return () => {                                                             
      if (channel) {                                                           
        channel.leave()                                                        
          .receive("ok", () => console.log("Left channel successfully"))       
          .receive("error", () => console.log("Error leaving channel"));       
      }                                                                        
      if (socket) {                                                            
        socket.disconnect(() => console.log("Socket disconnected"));           
      }                                                                        
    };                                                                         
  });                                                                          
</script>                                                                      
                                                                               
<div>                                                                          
  <h1>Stock Dashboard</h1>                                                     
  <p><strong>Socket Status:</strong> {socketStatus}</p>                        
  <p><strong>Channel Status:</strong> {channelStatus}</p>                      
  {#if channelResponse}                                                        
    <p><strong>Channel Response:</strong></p>                                  
    <pre>{JSON.stringify(channelResponse, null, 2)}</pre>                      
  {/if}                                                                        
</div>                                                                         
                                                                               
<style>                                                                        
  pre {                                                                        
    background-color: #f4f4f4;                                                 
    padding: 10px;                                                             
    border-radius: 4px;                                                        
  }                                                                            
</style>      
