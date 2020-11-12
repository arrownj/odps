---
keyword: edge table import
---

# Edge table import

This topic provides the sample code that is used to import an edge table.

Sample code:

```
import java.io.IOException;
import com.aliyun.odps.conf.Configuration;
import com.aliyun.odps.data.TableInfo;
import com.aliyun.odps.graph.ComputeContext;
import com.aliyun.odps.graph.GraphJob;
import com.aliyun.odps.graph.GraphLoader;
import com.aliyun.odps.graph.Vertex;
import com.aliyun.odps.graph.VertexResolver;
import com.aliyun.odps.graph.MutationContext;
import com.aliyun.odps.graph.VertexChanges;
import com.aliyun.odps.graph.Edge;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.WritableComparable;
import com.aliyun.odps.io.WritableRecord;
/**
 * This example describes how to compile a Graph job program to load data of different data types. It covers how GraphLoader
 * and VertexResolver are used together to build a graph.
 *
 * A MaxCompute Graph job uses MaxCompute tables as the input. For example, a job uses two tables as the input. One stores information about vertices, and the other stores information about edges.
 * Format of the table that stores information about vertices:
 * +------------------------+
 * | VertexID | VertexValue |
 * +------------------------+
 * |       id0|            9|
 * +------------------------+
 * |       id1|            7|
 * +------------------------+
 * |       id2|            8|
 * +------------------------+
 *
 * Format of the table that stores information about edges:
 * +-----------------------------------+
 * | VertexID | DestVertexID| EdgeValue|
 * +-----------------------------------+
 * |       id0|          id1|         1|
 * +-----------------------------------+
 * |       id0|          id2|         2|
 * +-----------------------------------+
 * |       id2|          id1|         3|
 * +-----------------------------------+
 *
 * The two tables show that id0 has two outgoing edges that point to id1 and id2. id2 has one outgoing edge that points to id1, and id1 has no outgoing edges.
 *
 * For data of this type, in GraphLoader::load(LongWritable, Record, MutationContext), MutationContext#addVertexRequest(Vertex) can be used to add vertices to the graph.
 * link MutationContext#addEdgeRequest(WritableComparable, Edge) can be used to add edges to the graph. In
 * link VertexResolver#resolve(WritableComparable, Vertex, VertexChanges, boolean),
 * vertices and edges added by using the load() method are combined to a vertex object. This object is used as the return value and added to the graph for computing.
 *
 **/
public class VertexInputFormat {
  private final static String EDGE_TABLE = "edge.table";
  /**
   * Resolve records to vertices and edges. Each record indicates a vertex or edge based on its source.
   * <p>
   * The following process is similar to the map stage of com.aliyun.odps.mapreduce.Mapper.
   * Enter a record to generate key-value pairs.
   * The keys are vertex IDs, and the values are vertices or edges that are written based on context. These key-value pairs are aggregated by using LoadingVertexResolver based on the vertex IDs.
   *
   * Note: The vertices or edges added here are requests sent based on the record content, and are not used in computing.
   * Only the vertices or edges added by using VertexResolver are used for computing.
   **/
  public static class VertexInputLoader extends
      GraphLoader<LongWritable, LongWritable, LongWritable, LongWritable> {
    private boolean isEdgeData;
    /**
     * Configure VertexInputLoader.
     *
     * @param conf
     *          Indicates the configured parameters of a job. These parameters are configured in the MAIN function of GraphJob or by running the SET command on the client.
     * @param workerId
     *          Indicates the serial number of the running worker. The number starts from 0 and can be used to build a unique vertex ID.
     * @param inputTableInfo
     *          Indicates the information about the input table that is loaded to the running worker. The information can be used to determine the data type of the input data (the format of the record).
     **/
    @Override
    public void setup(Configuration conf, int workerId, TableInfo inputTableInfo) {
      isEdgeData = conf.get(EDGE_TABLE).equals(inputTableInfo.getTableName());
    }
    /**
     * Resolve the record to edges based on the record content and send a request to add them to the graph.
     *
     * @param recordNum
     *          Indicates the serial number of the record. The number starts from 1 and is separately counted in each worker.
     * @param record
     *          Indicates the records in the input table. The table contains three columns, which indicate the source vertex, destination vertex, and edge weight.
     * @param context
     *          Indicates the context. The context is used when you send a request to add resolved edges to the graph.
     **/
    @Override
    public void load(
        LongWritable recordNum,
        WritableRecord record,
        MutationContext<LongWritable, LongWritable, LongWritable, LongWritable> context)
        throws IOException {
      if (isEdgeData) {
        /**
         * Data comes from the table that stores information about edges.
         *
         * 1. The first column indicates the IDs of source vertices.
         **/
        LongWritable sourceVertexID = (LongWritable) record.get(0);
        /**
         * 2. The second column indicates the IDs of destination vertices.
         **/
        LongWritable destinationVertexID = (LongWritable) record.get(1);
        /**
         * 3. The third column indicates edge weights.
         **/
        LongWritable edgeValue = (LongWritable) record.get(2);
        /**
         * 4. Create an edge based on a destination vertex ID and an edge weight.
         **/
        Edge<LongWritable, LongWritable> edge = new Edge<LongWritable, LongWritable>(
            destinationVertexID, edgeValue);
        /**
         * 5. Send a request to add the edge to a source vertex.
         **/
        context.addEdgeRequest(sourceVertexID, edge);
        /**
         * 6. If each record indicates a bidirectional edge, repeat Step 4 and Step 5.
          * Edge<LongWritable, LongWritable> edge2 = new
         * Edge<LongWritable, LongWritable>( sourceVertexID, edgeValue);
         * context.addEdgeRequest(destinationVertexID, edge2);
         **/
      } else {
        /**
         * Data comes from the table that stores information about vertices.
         *
         * 1. The first column indicates the IDs of vertices.
         **/
        LongWritable vertexID = (LongWritable) record.get(0);
        /**
         * 2. The second column indicates the values of vertices.
         **/
        LongWritable vertexValue = (LongWritable) record.get(1);
        /**
         * 3. Create a vertex based on an ID and a value.
         **/
        MyVertex vertex = new MyVertex();
        /**
         * 4. Initialize the vertex.
         **/
        vertex.setId(vertexID);
        vertex.setValue(vertexValue);
        /**
         * 5. Send a request to add the vertex.
         **/
        context.addVertexRequest(vertex);
      }
    }
  }
  /**
   * Summarize key-value pairs generated by using GraphLoader::load(LongWritable, Record, MutationContext).
   * This process is similar to the reduce stage of com.aliyun.odps.mapreduce.Reducer. For a unique vertex ID, all actions, such as 
   * adding or removing vertices or edges for the ID, are stored in VertexChanges.
   *
   * Note: Not only conflicting vertices or edges added by using the load() method are called. A conflict occurs when multiple same vertex objects or duplicate edges are added.
   * All the IDs that are requested to be generated by using the load() method are called.
   **/
  public static class LoadingResolver extends
      VertexResolver<LongWritable, LongWritable, LongWritable, LongWritable> {
    /**
     * Process a request to add or remove vertices or edges for an ID.
     *
     * VertexChanges has four APIs, which correspond to the four APIs of MutationContext:
     * VertexChanges::getAddedVertexList() corresponds to
     * MutationContext::addVertexRequest(Vertex).
     * In the load() method, if a request is sent to add vertex objects with the same ID, the objects are collected to the returned list.
     * VertexChanges::getAddedEdgeList() corresponds to 
     * MutationContext::addEdgeRequest(WritableComparable, Edge).
     * If a request is sent to add edge objects with the same source vertex ID, the objects are collected to the returned list.
     * VertexChanges::getRemovedVertexCount() corresponds to 
     * MutationContext::removeVertexRequest(WritableComparable).
     * If a request is sent to remove vertices with the same ID, the number of total removal requests is returned.
     * VertexChanges#getRemovedEdgeList() corresponds to 
     * MutationContext#removeEdgeRequest(WritableComparable, WritableComparable).
     * If a request is sent to remove edge objects with the same source vertex ID, the objects are collected to the returned list.
     *
     * You can process the changes in the ID and state whether the ID is used in computing in the return value.
     * If the returned vertex is not null, the ID is used in subsequent computing. If the returned vertex is null, the ID is not used in subsequent computing.
     *
     * @param vertexId
     *          Indicates the ID of the vertex that is requested to be added or the source vertex ID of the edge that is requested to be added.
     * @param vertex
     *          Indicates an existing vertex object. The value of this parameter is always null in the data loading phase.
     * @param vertexChanges
     *          Indicates a collection of vertices or edges that are requested to be added or removed for the ID.
     * @param hasMessages
     *          Indicates whether messages are sent to the ID. The value of this parameter is always false in the data loading phase.
     **/
    @Override
    public Vertex<LongWritable, LongWritable, LongWritable, LongWritable> resolve(
        LongWritable vertexId,
        Vertex<LongWritable, LongWritable, LongWritable, LongWritable> vertex,
        VertexChanges<LongWritable, LongWritable, LongWritable, LongWritable> vertexChanges,
        boolean hasMessages) throws IOException {
      /**
       * 1. Obtain the vertex object for computing.
       **/
      MyVertex computeVertex = null;
      if (vertexChanges.getAddedVertexList() == null
          || vertexChanges.getAddedVertexList().isEmpty()) {
        computeVertex = new MyVertex();
        computeVertex.setId(vertexId);
      } else {
        /**
         * Each record indicates a unique vertex in the table that stores information about vertices.
         **/
        computeVertex = (MyVertex) vertexChanges.getAddedVertexList().get(0);
      }
      /**
       * 2. Add the edge, which is requested to be added to the vertex, to the vertex object. If the data is a possible duplicate, deduplicate it based on algorithm requirements.
       **/
      if (vertexChanges.getAddedEdgeList() ! = null) {
        for (Edge<LongWritable, LongWritable> edge : vertexChanges
            .getAddedEdgeList()) {
          computeVertex.addEdge(edge.getDestVertexId(), edge.getValue());
        }
      }
      /**
       * 3. Return the vertex object and add it to the final graph for computing.
       **/
      return computeVertex;
    }
  }
  /**
   * Determine the actions of the vertex that is used in computing.
   *
   **/
  public static class MyVertex extends
      Vertex<LongWritable, LongWritable, LongWritable, LongWritable> {
    /**
     * Write the edge of the vertex to the result table based on the format of the input table. Make sure that the formats and data of the input and output tables are the same.
     *
     * @param context
     *          Indicates the runtime context.
     * @param messages
     *          Indicates the input messages.
     **/
    @Override
    public void compute(
        ComputeContext<LongWritable, LongWritable, LongWritable, LongWritable> context,
        Iterable<LongWritable> messages) throws IOException {
      /**
       * Write the ID and value of the vertex to the result table that stores information about vertices.
       **/
      context.write("vertex", getId(), getValue());
      /**
       * Write the edge of the vertex to the result table that stores information about edges.
       **/
      if (hasEdges()) {
        for (Edge<LongWritable, LongWritable> edge : getEdges()) {
          context.write("edge", getId(), edge.getDestVertexId(),
              edge.getValue());
        }
      }
      /**
       * Perform only one iteration.
       **/
      voteToHalt();
    }
  }
  /**
   * @param args
   * @throws IOException
   */
  public static void main(String[] args) throws IOException {
    if (args.length < 4) {
      throw new IOException(
          "Usage: VertexInputFormat <vertex input> <edge input> <vertex output> <edge output>");
    }
    /**
     * Use GraphJob to configure a Graph job.
     */
    GraphJob job = new GraphJob();
    /**
     * 1. Specify input graph data and the table that stores information about edges.
     */
    job.addInput(TableInfo.builder().tableName(args[0]).build());
    job.addInput(TableInfo.builder().tableName(args[1]).build());
    job.set(EDGE_TABLE, args[1]);
    /**
     * 2. Specify the data loading mode and resolve the records to edges. This process is similar to the map stage. The generated key is the vertex ID, and the generated value is the edge.
     */
    job.setGraphLoaderClass(VertexInputLoader.class);
    /**
     * 3. Specify the data loading phase to generate the vertex that is used in computing. This process is similar to the reduce stage. In the reduce stage, edges that are generated in the map stage are combined into a vertex.
     */
    job.setLoadingVertexResolverClass(LoadingResolver.class);
    /**
     * 4. Specify the actions of the vertex that is used in computing. The vertex.compute() method is used for each iteration.
     */
    job.setVertexClass(MyVertex.class);
    /**
     * 5. Specify the result table of the Graph job and write the computing results to the result table.
     */
    job.addOutput(TableInfo.builder().tableName(args[2]).label("vertex").build());
    job.addOutput(TableInfo.builder().tableName(args[3]).label("edge").build());
    /**
     * 6. Submit the job for execution.
     */
    job.run();
  }
}
            
```

