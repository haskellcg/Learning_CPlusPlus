## 模板基类的成员访问
  * [学习C++模板](https://blog.csdn.net/Windgs_YF/article/details/80927091)

## spdlog的输出
  * [spdlog的输出格式详解](https://www.bilibili.com/read/cv15130912)
  ```
  // multi sinks
  void multi_sink_example()
  {
      auto console_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
      console_sink->set_level(spdlog::level::warn);
      console_sink->set_pattern("[multi_sink_example] [%^%l%$] %v");

      auto file_sink = std::make_shared<spdlog::sinks::basic_file_sink_mt>("logs/multisink.txt", true);
      file_sink->set_level(spdlog::level::trace);

      spdlog::logger logger("multi_sink", {console_sink, file_sink});
      logger.set_level(spdlog::level::debug);
      logger.warn("this should appear in both console and file");
      logger.info("this message should not appear in the console, only in the file");
  }
  ```
