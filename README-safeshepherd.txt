This is how I upgraded the Safeshepherd build of Spark from Spark 1.0.0 to Spark 1.0.2.

-- when moving up to a new spark version, I highly recommend checking out a new copy of whenceforth/spark rather than working in the existing local copy.
This makes it much easier to throw things out if something goes wrong.

--in your new local copy of whenceforth/spark, make sure you have the original project configured as an upstream.

git remote -v should list it.
If not, do this:


git remote add upstream https://github.com/apache/spark.git


See https://help.github.com/articles/configuring-a-remote-for-a-fork for more info


--fetch upstream into whenceforth/spark fork:
git fetch upstream

-- checkout the latest safe shepherd version tag
git checkout v1.0.0-safeshepherd-d

-- make a new branch from there for the merge
git checkout -b merge102

-- merge the upstream tag
git merge v1.0.2

-- resolve conflicts && commit

-- build our custom jar


export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"

mvn -Dhadoop.version=2.0.0-mr1-cdh4.2.0 -DskipTests clean package

This will build the assembly into subdirectory assembly/target/scala-2.10

-- put the custom jar into S3, in the bucket com.safeshepherd.builds
Note that the spark 1.0.0 custom jar is also there.

To change the jar file used by the non-streaming shepbot, update the JAR_FILE variable in 


meloncard/stack/fleet/spark_control/master-scripts/pull-custom-spark-jar.sh
(Note: that file is in branch spark_control_prelim)


pull-custom-spark-jar.sh is included in the Docker image, so AFTER committing the update to pull-custom-spark-jar.sh, 
rebuild the docker image using 
meloncard/stack/fleet/spark_control/build-docker-image.sh

(build-docker-image.sh includes the output of git describe in the Docker image.)

TO RUN LOCALLY WITH THE NEW VERSION:

-- export OLD_SPARK_HOME=$SPARK_HOME
-- update your SPARK_HOME environment variable to point at your new location for whenceforth/spark.

-- mkdir -p $SPARK_HOME/lib
-- copy the new assembly jar into $SPARK_HOME/lib
   (You probably also want to cp -i $OLD_SPARK_HOME/conf/log4j.properties $SPARK_HOME/conf

-- After doing a successful local run, merge to whenceforth/spark and push to origin.
Then create an annotated tag and push the tag to github with 
git push --tags



