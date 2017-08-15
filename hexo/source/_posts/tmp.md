import java.io.IOException;
import java.nio.channels.ClosedChannelException;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Processor {
	private Selector sel;
	private static final ExecutorService es = Executors.newFixedThreadPool(2 * Runtime.getRuntime().availableProcessors());
	
	Processor() throws IOException{
		this.sel = Selector.open();
	}
	
	public void register(SocketChannel sc, int key) throws ClosedChannelException{
		sc.register(sel, key);
	}
	
	private void start() {
		//提交Callable任务
		es.submit(() -> {
			while(!Thread.currentThread().isInterrupted()){  
		            // 该调用会阻塞，直到至少有一个事件发生  
		            try {
						sel.select();
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}   
		            Set<SelectionKey> keys = sel.selectedKeys();  
		            Iterator<SelectionKey> iter = keys.iterator();  
		            while (iter.hasNext()) {   
		                SelectionKey key = (SelectionKey) iter.next();   
		                iter.remove();   
		                //process(key);   
		            }   
			}
		});
	}
}
