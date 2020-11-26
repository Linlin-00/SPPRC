#windows下运行没有问题的代码，在ubuntu下运行出错
编译后，运行./demo报错
terminate called after throwing an instance of 'std::runtime_error'
  what():  cannot open file access.xml
Aborted

找到对应的
rapidxml_utils.hpp文件，其中代码为：
file(const char* filename)
        {
            using namespace std;

            // Open stream
            basic_ifstream<Ch> stream(filename, ios::binary);
            if (!stream)
                throw runtime_error(string("cannot open file ") + filename);
            stream.unsetf(ios::skipws);

            // Determine stream size
            stream.seekg(0, ios::end);
            size_t size = stream.tellg();
            stream.seekg(0);

            // Load data and add terminating 0
            m_data.resize(size + 1);
            stream.read(&m_data.front(), static_cast<streamsize>(size));
            m_data[size] = 0;
        }


问题为：
在ubuntu下，xml文件放进xml文件夹了，但是代码里面忘记修改相应的路径

access.xml 改为../xml/access.xml
