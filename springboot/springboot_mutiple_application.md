# SpringBoot配置多个application.ptoperties文件.md

Step 1. Create file src/main/resources/META-INF/spring.factories with the following:

~~~
org.springframework.boot.env.EnvironmentPostProcessor=\
com.example.YourEnvironmentPostProcessor
Step 2. As an example create a file src/main/resources/custom.properties with:
~~~

server.port=8081
Step 3. Now create you post processor class

~~~
package com.example;

public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

  private final PropertiesPropertySourceLoader loader = new PropertiesPropertySourceLoader();

  @Override
  public void postProcessEnvironment(ConfigurableEnvironment environment,
                                     SpringApplication application) {
    Resource path = new ClassPathResource("custom.properties");

    PropertySource<?> propertySource = loadProps(path);
    environment.getPropertySources().addFirst(propertySource);

    PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

    Resource[] configResources = resolver.getResources("classpath:prop/*.properties");

    for(Resource resource : configResources) {
      List<PropertySource<?>> psList = this.loadProps(resource);
      psList.forEach(action -> {
          environment.getPropertySources().addLast(action);
        });
    }
  }

  private List<PropertySource<?>> loadProps(Resource path) {
    if (!path.exists()) {
      throw new IllegalArgumentException("Resource " + path + " does not exist");
    }
    try {
      return this.loader.load(path.getFilename(), path, null);
    }
    catch (IOException ex) {
      throw new IllegalStateException(
          "Failed to load props configuration from " + path, ex);
    }
  }

}
~~~

