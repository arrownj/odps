---
keyword: [PageRank, web page ranking]
---

# PageRank

PageRank is an algorithm used to rank web pages. The input of PageRank is a directed graph. Each vertex represents a web page. Each edge represents a link between two web pages. If the web pages are not connected, no edge exists between the vertices.

How the PageRank algorithm works:

-   Initialization: A vertex value indicates the rank value of PageRank. The rank value is of the DOUBLE type. During initialization, the value of each vertex is `1/TotalNumVertices`.
-   Iteration formula:

    `PageRank(i) = 0.15/TotalNumVertices + 0.85 × Value of sum`

    sum is used to add up `PageRank(j)/out_degree(j)` of each vertex that points to center i. j in the formula refers to each vertex that points to center i.


The PageRank algorithm is suitable for MaxCompute Graph programs. Each vertex j maintains its PageRank value and sends `PageRank(j)/out_degree(j)` to its adjacent vertices \(for voting\) in each iteration. In the next iteration, each vertex recalculates the PageRank value by using the iteration formula.

## Sample code

```
import java.io.IOException;

import org.apache.log4j.Logger;

import com.aliyun.odps.io.WritableRecord;
import com.aliyun.odps.graph.ComputeContext;
import com.aliyun.odps.graph.GraphJob;
import com.aliyun.odps.graph.GraphLoader;
import com.aliyun.odps.graph.MutationContext;
import com.aliyun.odps.graph.Vertex;
import com.aliyun.odps.graph.WorkerContext;
import com.aliyun.odps.io.DoubleWritable;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.NullWritable;
import com.aliyun.odps.data.TableInfo;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.io.Writable;

public class PageRank {

  private final static Logger LOG = Logger.getLogger(PageRank.class);

  public static class PageRankVertex extends
      Vertex<Text, DoubleWritable, NullWritable, DoubleWritable> {

    @Override
    public void compute(
        ComputeContext<Text, DoubleWritable, NullWritable, DoubleWritable> context,
        Iterable<DoubleWritable> messages) throws IOException {
      if (context.getSuperstep() == 0) {
        setValue(new DoubleWritable(1.0 / context.getTotalNumVertices()));
      } else if (context.getSuperstep() >= 1) {
        double sum = 0;
        for (DoubleWritable msg : messages) {
          sum += msg.get();
        }
        DoubleWritable vertexValue = new DoubleWritable(
            (0.15f / context.getTotalNumVertices()) + 0.85f * sum);
        setValue(vertexValue);
      }
      if (hasEdges()) {
        context.sendMessageToNeighbors(this, new DoubleWritable(getValue()
            .get() / getEdges().size()));
      }
    }

    @Override
    public void cleanup(
        WorkerContext<Text, DoubleWritable, NullWritable, DoubleWritable> context)
        throws IOException {
      context.write(getId(), getValue());
    }
  }

  public static class PageRankVertexReader extends
      GraphLoader<Text, DoubleWritable, NullWritable, DoubleWritable> {

    @Override
    public void load(
        LongWritable recordNum,
        WritableRecord record,
        MutationContext<Text, DoubleWritable, NullWritable, DoubleWritable> context)
        throws IOException {
      PageRankVertex vertex = new PageRankVertex();
      vertex.setValue(new DoubleWritable(0));
      vertex.setId((Text) record.get(0));
      System.out.println(record.get(0));

      for (int i = 1; i < record.size(); i++) {
        Writable edge = record.get(i);
        System.out.println(edge.toString());
        if (!( edge.equals(NullWritable.get()))) {
          vertex.addEdge(new Text(edge.toString()), NullWritable.get());
        }
      }
      LOG.info("vertex edgs size: "
          + (vertex.hasEdges() ? vertex.getEdges().size() : 0));
      context.addVertexRequest(vertex);
    }

  }

  private static void printUsage() {
    System.out.println("Usage: <in> <out> [Max iterations (default 30)]");
    System.exit(-1);
  }

  public static void main(String[] args) throws IOException {
    if (args.length < 2)
      printUsage();

    GraphJob job = new GraphJob();

    job.setGraphLoaderClass(PageRankVertexReader.class);
    job.setVertexClass(PageRankVertex.class);
    job.addInput(TableInfo.builder().tableName(args[0]).build());
    job.addOutput(TableInfo.builder().tableName(args[1]).build());

    // default max iteration is 30
    job.setMaxIteration(30);
    if (args.length >= 3)
      job.setMaxIteration(Integer.parseInt(args[2]));

    long startTime = System.currentTimeMillis();
    job.run();
    System.out.println("Job Finished in "
        + (System.currentTimeMillis() - startTime) / 1000.0 + " seconds");
  }
}
            
```

Description:

-   Row 23: Define the PageRankVertex class.
    -   The vertex value indicates the current PageRank value of the vertex \(web page\).
    -   The compute\(\) method uses the following iteration formula to update the vertex value: `PageRank(i) = 0.15/TotalNumVertices + 0.85 × Value of sum`
    -   The cleanup\(\) method writes the vertex and its PageRank value to the result table.
-   Row 55: Define the PageRankVertexReader class, load a graph, and resolve each record in the table into a vertex. The first column of the table is the source vertices and other columns are the destination vertices.
-   Row 88: Include the main function, define the GraphJob class, and specify the maximum number of iterations, the input and output tables, and the implementation of Vertex and GraphLoader. By default, a maximum of 30 iterations can be performed.

