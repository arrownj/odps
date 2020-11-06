---
keyword: SSSP
---

# SSSP

Dijkstra's algorithm is a common algorithm that is used to calculate the Single Source Shortest Path \(SSSP\) in a directed graph.

How the Dijkstra's algorithm works:

-   Initialization: The path from s to s is 0 \(d\[s\] = 0\), and the path from u to s is infinite \(d\[u\] = ∞\).
-   Iteration: If an edge from u to v exists, the shortest path from s to v is updated to `d[v] = min(d[v], d[u] + weight(u, v))`. The iteration does not end until the paths from all vertices to s do not change.

    **Note:** Shortest path: For a weighted directed graph `G = (V,E)`, multiple paths are available from source vertex s to sink vertex v. The path with the smallest sum of edge weights is called the shortest path from s to v.


The working mode of the algorithm shows that the algorithm is suitable for MaxCompute Graph. Each vertex maintains the current shortest path to the source vertex. If the path changes, the new path is added with the edge weight, and a message is sent to notify adjacent vertices. In the next iteration, the adjacent vertices update the shortest paths based on the received message. If the shortest path between each vertex and the source vertex does not change, the iteration ends.

## Sample code

Example:

```
import java.io.IOException;
import com.aliyun.odps.io.WritableRecord;
import com.aliyun.odps.graph.Combiner;
import com.aliyun.odps.graph.ComputeContext;
import com.aliyun.odps.graph.Edge;
import com.aliyun.odps.graph.GraphJob;
import com.aliyun.odps.graph.GraphLoader;
import com.aliyun.odps.graph.MutationContext;
import com.aliyun.odps.graph.Vertex;
import com.aliyun.odps.graph.WorkerContext;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.data.TableInfo;

public class SSSP {
  public static final String START_VERTEX = "sssp.start.vertex.id";
  public static class SSSPVertex extends
      Vertex<LongWritable, LongWritable, LongWritable, LongWritable> {
    private static long startVertexId = -1;
    public SSSPVertex() {
      this.setValue(new LongWritable(Long.MAX_VALUE));
    }

    public boolean isStartVertex(
        ComputeContext<LongWritable, LongWritable, LongWritable, LongWritable> context) {
      if (startVertexId == -1) {
        String s = context.getConfiguration().get(START_VERTEX);
        startVertexId = Long.parseLong(s);
      }
      return getId().get() == startVertexId;
    }

    @Override
    public void compute(
        ComputeContext<LongWritable, LongWritable, LongWritable, LongWritable> context,
        Iterable<LongWritable> messages) throws IOException {
      long minDist = isStartVertex(context) ? 0 : Integer.MAX_VALUE;
      for (LongWritable msg : messages) {
        if (msg.get() < minDist) {
          minDist = msg.get();
        }
      }

      if (minDist < this.getValue().get()) {
        this.setValue(new LongWritable(minDist));
        if (hasEdges()) {
          for (Edge<LongWritable, LongWritable> e : this.getEdges()) {
            context.sendMessage(e.getDestVertexId(), new LongWritable(minDist
                + e.getValue().get()));
          }
        }
      } else {
        voteToHalt();
      }
    }

    @Override
    public void cleanup(
        WorkerContext<LongWritable, LongWritable, LongWritable, LongWritable> context)
        throws IOException {
      context.write(getId(), getValue());
    }
  }

  public static class MinLongCombiner extends
      Combiner<LongWritable, LongWritable> {

    @Override
    public void combine(LongWritable vertexId, LongWritable combinedMessage,
        LongWritable messageToCombine) throws IOException {
      if (combinedMessage.get() > messageToCombine.get()) {
        combinedMessage.set(messageToCombine.get());
      }
    }

  }

  public static class SSSPVertexReader extends
      GraphLoader<LongWritable, LongWritable, LongWritable, LongWritable> {

    @Override
    public void load(
        LongWritable recordNum,
        WritableRecord record,
        MutationContext<LongWritable, LongWritable, LongWritable, LongWritable> context)
        throws IOException {
      SSSPVertex vertex = new SSSPVertex();
      vertex.setId((LongWritable) record.get(0));
      String[] edges = record.get(1).toString().split(",");
      for (int i = 0; i < edges.length; i++) {
        String[] ss = edges[i].split(":");
        vertex.addEdge(new LongWritable(Long.parseLong(ss[0])),
            new LongWritable(Long.parseLong(ss[1])));
      }

      context.addVertexRequest(vertex);
    }

  }

  public static void main(String[] args) throws IOException {
    if (args.length < 2) {
      System.out.println("Usage: <startnode> <input> <output>");
      System.exit(-1);
    }

    GraphJob job = new GraphJob();
    job.setGraphLoaderClass(SSSPVertexReader.class);
    job.setVertexClass(SSSPVertex.class);
    job.setCombinerClass(MinLongCombiner.class);

    job.set(START_VERTEX, args[0]);
    job.addInput(TableInfo.builder().tableName(args[1]).build());
    job.addOutput(TableInfo.builder().tableName(args[2]).build());

    long startTime = System.currentTimeMillis();
    job.run();
    System.out.println("Job Finished in "
        + (System.currentTimeMillis() - startTime) / 1000.0 + " seconds");
  }
}
                
```

Description:

-   Row 19: Define `SSSPVertex`.
    -   The vertex value indicates the current shortest path from this vertex to the `startVertexId` source vertex.
    -   The `compute()` method uses the `d[v] = min(d[v], d[u] + weight(u, v))` iteration formula to update the vertex value.
    -   The `cleanup()` method writes the vertex and its shortest path to the source vertex to the result table.
-   Row 58: If the vertex value does not change, `voteToHalt()` is called to notify the framework that this vertex enters the halt state. The calculation ends after all vertices enter the halt state.
-   Row 70: Define `MinLongCombiner` and combine messages that are sent to the same vertex to optimize performance and reduce memory usage.
-   Row 83: Define the `SSSPVertexReader` class, load a graph, and parse each record in the table into a vertex. The first column of the record is the vertex ID, and the second column stores all edge sets that start from the vertex, such as 2:2, 3:1, and 4:4.
-   Row 106: Include the `main` function, define `GraphJob`, and specify input and output tables and the implementation of `Vertex`, `GraphLoader`, and `Combiner`.

